---
https://developers.prtimes.jp/2024/12/24/react-router-v7-upgrade/
---

# React Router v7にバージョンアップしました



こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

先日のリファクタリングデーでReact Routerのバージョンをv6からv7にバージョンアップしました。その際に行った変更やPR TIMESにおけるReact Router v7の使い方についてご紹介します。リファクタリングデーについては以下の記事をご参照ください。

あわせて読みたい

目次

## バージョンアップの手順

元々PR TIMESではReact Routerをv5.3.0から使用しており、v6（v6.3.0）へのバージョンアップは2年前に完了していました。しかし、v6.4.0にバージョンアップを行う際にソフトナビゲーション（SPA遷移）を止める処理でBreaking Changeが発生し、長らくバージョンアップができていませんでした。そのため、いきなりv7へバージョンアップをするのではなく、段階を踏んでバージョンアップを行いました。

ソフトナビゲーションを止める処理は、フォームに対して何か入力を行った後、ページ遷移またはリロードを行った際に確認ダイアログを表示する処理で必要となります。

### v6.3.0からv6.26.2へのバージョンアップ

まず、v6.3.0からv6.26.2（バージョンアップを行った時の最新バージョン）にアップデートしました。このバージョンアップではソフトナビゲーションの処理とルーティング定義のコードを修正しました。

v6.3.0時点でのソフトナビゲーションを止めるコードは以下のようになっていました。

```
import type { Blocker, History, Transition } from 'history';
import { useCallback, useContext, useEffect, useState } from 'react';
import {
  useLocation,
  useNavigate,
  UNSAFE_NavigationContext as NavigationContext,
} from 'react-router-dom';

const MESSAGE =
  '本当にページを移動してもよろしいですか？入力内容が破棄されます';

export const usePageTransitionBlocker = () => {
  const [shouldBlock, setShouldBlock] = useState(false);

  const navigate = useNavigate();
  const location = useLocation();
  const [lastTransition, setLastTransition] = useState<Transition>();
  const [hasNavigationConfirmed, setHasNavigationConfirmed] = useState(false);

  const handleBlockedNavigation = useCallback<Blocker>(
    (transition) => {
      if (
        !hasNavigationConfirmed &&
        transition.location.pathname !== location.pathname
      ) {
        setLastTransition(transition);
        if (window.confirm(MESSAGE)) {
          setHasNavigationConfirmed(true);
        }

        return false;
      }
    },
    [hasNavigationConfirmed, location.pathname],
  );

  useEffect(() => {
    if (hasNavigationConfirmed && lastTransition) {
      navigate(lastTransition.location.pathname);
    }
  }, [hasNavigationConfirmed, lastTransition, navigate]);

  /** @see https://github.com/remix-run/react-router/issues/8139#issuecomment-954431589 */
  // navigator.block プロパティが型定義上省略されているため as を使用
  // 現在のreact-router実装の内部でblockが使用されていないために消されているだけで、実装上存在している
  const navigator = useContext(NavigationContext).navigator as History;

  useEffect(() => {
    if (!shouldBlock) return;

    const unblock = navigator.block((transition) =>
      handleBlockedNavigation({
        retry: () => {
          unblock();
          transition.retry();
        },
      }),
    );

    return unblock;
  }, [handleBlockedNavigation, navigator, shouldBlock]);

  const onWindowOrTabClose = useCallback(
    (e: BeforeUnloadEvent) => {
      if (!shouldBlock) return;

      e.returnValue = MESSAGE;
      return MESSAGE;
    },
    [shouldBlock],
  );

  useEffect(() => {
    window.addEventListener('beforeunload', onWindowOrTabClose);

    return () => {
      window.removeEventListener('beforeunload', onWindowOrTabClose);
    };
  }, [onWindowOrTabClose]);

  return {
    shouldBlock,
    setShouldBlock,
  };
};
```

v5からv6.3.0にアプデートを行う際に [Prompt](https://v5.reactrouter.com/core/api/Prompt) が削除されており、ハック（navigatorの型をHistoryに強制してblockを使用）しなければソフトナビゲーションを止めることはできませんでした。この問題は issue にも挙がっており、v5からv6にバージョンアップができないユーザーが多かったと思います。

GitHub

その後様々な議論があったのち、v6.7.0で unstable で [useBlocker](https://reactrouter.com/6.28.1/hooks/use-blocker) が追加され、さらにその後 [unstable\_usePrompt](https://reactrouter.com/6.28.1/hooks/use-prompt) が追加され、ソフトナビゲーションを止めることができるようになりました。そのため、v6.26.2にアップデートする際に上記のコードを以下のように修正しました。[useBeforeUnload](https://reactrouter.com/6.28.1/hooks/use-before-unload) と unstable\_usePrompt を使用することでコードがかなりシンプルになりました。

```
import {useCallback} from 'react';
import {useBeforeUnload, unstable_usePrompt} from 'react-router';

type Options = {
  message: string;
  beforeNavigate?: () => void;
};

export const usePageTransitionBlocker = (
  {message, beforeNavigate}: Options = {
    message: '本当にページを移動してもよろしいですか？入力内容が破棄されます',
  },
) => {
  const [shouldBlock, setShouldBlock] = useState(false);

  // ハードナビゲーション（aタグでの遷移やリロードなどを止める）
  useBeforeUnload(
    useCallback(
      (event) => {
        if (!shouldBlock) {
          return;
        }

        if (window.confirm(message)) {
          if (beforeNavigate !== undefined) {
            beforeNavigate();
          }

          setShouldBlock(false);
        } else {
          event.preventDefault();
          event.returnValue = '';
        }
      },
      [shouldBlock, message, beforeNavigate, setShouldBlock],
    ),
  );

  // ソフトナビゲーション（Linkコンポーネントでの遷移などを止める）
  unstable_usePrompt({
    message,
    when: ({currentLocation, nextLocation}) =>
      shouldBlock && currentLocation.pathname !== nextLocation.pathname,
  });

  return {
    shouldBlock,
    setShouldBlock,
  };
};
```

バージョンアップにあたり、この変更に加えてルート定義を変更する必要がありました。元々ルート定義には [BrowserRouterコンポーネント](https://reactrouter.com/6.28.1/router-components/browser-router) を使用していましたが、unstable\_usePrompt を使用するためには新しい RouterProvider を使用する必要がありました。そのために以下のようにコードを修正しました。

```
// before
export function Router() {
  return (
    <BrowserRouter>
      <Routes>
        <Route
          path='/'
          element={<TopPage />}
        />
        <Route
          path='/test'
          element={<TestPage />}
        />
      </Routes>
    </BrowserRouter>
  );
}

// after
const router = createBrowserRouter(
  [
    {
      path: '/',
      element: <TopPage />
    },
    {
      path: '/test',
      element: <TestPage />
    }
  ],
  {
    future: {
      v7_fetcherPersist: true,
      v7_normalizeFormMethod: true,
      v7_partialHydration: true,
      v7_relativeSplatPath: true,
      v7_skipActionErrorRevalidation: true,
    },
  },
);

export function Router() {
  return <RouterProvider router={router} />
}
```

また新しい RouterProvider では v7 の変更を事前に利用するための Future Flags が提供されていたため、この段階で全てのFlagをオンにしておきました。

あわせて読みたい

### v6.26.2からv7へのバージョンアップ

v6.26.2からv7へのバージョンアップはとても少ない変更で完了することができました。行ったことはパッケージのrenameとuseNavigateが非同期になったことに対するリントエラーの対応のみです。

これまで React Router を使用するときには [react-router-dom](https://www.npmjs.com/package/react-router-dom) を使用していましたが、React Router v7からは [react-router](https://www.npmjs.com/package/react-router) を使用します。そのため、package.jsonと各import元を `react-router-dom` から `react-router` に変更しました。

あわせて読みたい

また、useNavigate が非同期になったことにより、[no-floating-promises](https://typescript-eslint.io/rules/no-floating-promises/) のルールでエラーが発生したため、 void 演算子をつけて対応を行いました。

```
export function Hoge() {
  const navigate = useNavigate();
  const onClick = () => {
-   navigate(-1);
+   void navigate(-1);
  }

  return <button onClick={onClick}>戻る</button>
}
```

上記の変更だけでv7へのアップデートを完了することができました。これは一度v6.26.2へのバージョンアップを挟んだことによるもので、戦略的にバージョンアップをする重要性を感じました。

## Library mode or Framework mode

React Router v7はLibrary modeとFramework modeの2種類があります。Framework modeを使用するメリットとしてページ単位に最適化されるcode splittingなどが挙げられます。しかし、PR TIMESでは元々 React Router を v5 から使用していたこともあり React Router に対する期待はルーティングライブラリであることにとどまっています。そのため、v7にアップデートした後も Library mode で使用しています。

あわせて読みたい

また、PR TIMESの現状の構成では Vite で行っている処理は JavaScript と CSS のバンドルのみで HTML の出力は行っていません。Framework mode を使用すると HTML の出力まで Vite で行うことになり、デプロイの仕組みなどを変更する必要があります。そのため、今後も Framework mode を使用することは考えていません。

## まとめ

今回 React Router v7 にバージョンアップを行いました。一番辛かった v5 から v6 のバージョンアップを考えると、v6 から v7はかなりスムーズにアップデートすることができました。今後 React Router は Future Flags を活用してバージョンアップをしていくことを表明しているので今後のバージョンアップもスムーズにいけるのではないかと予想しています。

あわせて読みたい

また、PR TIMESのフロントエンドチームではリファクタリングデーを活用して、ライブラリのバージョンをできるだけ最新の状態に保つ活動を行っています。フロントエンドはライブラリの更新頻度や流行り廃りが激しいですが、今後もリファクタリングデーなどを通じて追従していきたいと思います。

## **We are hiring!**

フロントエンドエンジニアを含む各種ポジションでの採用を進めています！興味があればぜひご応募ください。

株式会社PR TIMES

株式会社PR TIMESでは現在03-4. 開発部 フロントエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
