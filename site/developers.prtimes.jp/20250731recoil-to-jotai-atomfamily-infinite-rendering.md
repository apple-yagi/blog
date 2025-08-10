---
<https://developers.prtimes.jp/2025/07/31/recoil-to-jotai-atomfamily-infinite-rendering/>
---

# RecoilからJotaiのatomFamilyに移行したら無限レンダリングが発生した話

こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

先日、企業ページを Recoil から脱却したエントリーを公開しました。

あわせて読みたい


[企業ページにおけるRecoilからの段階的移行](https://developers.prtimes.jp/2025/07/18/company-page-recoil-migration/)

このエントリーでも紹介した通り、Recoil を useMemo、useContext、TanStack Query、Jotai に移行しました。その後、Recoil の atomFamily を Jotai の atomFamily に移行したことによる無限レンダリングの事象を発見し、修正したのでご紹介します。

目次

## 発生した事象

Jotai の atomFamily へ Recoil の atomFamily から移行した際、特定の条件下でコンポーネントが無限に再レンダリングされる問題が発生しました。

具体的には、以下のように atomFamily の引数としてオブジェクトリテラルを直接渡していたところ、該当のコンポーネントが再レンダリングされ続ける状態となりました。

```typescript
import {atomFamily} from 'jotai/utils';
import {atom} from 'jotai';

type PagerParameters = {
  displayType: 'pc' | 'sp';
  contentsId: ContentType;
  maxNumberPerPage: number;
};

export const pagerState = atomFamily((parameters: PagerParameters) =>
  atom<PagerState>((get) => {
    const currentPageNumber = get(
      currentPageNumberState(parameters.contentsId),
    );

    switch (parameters.displayType) {
      case 'pc': {
        return {
          currentPageNum: currentPageNumber,
          startIndex: (currentPageNumber - 1) * maxNumberPerPage.pc,
          endIndex: currentPageNumber * maxNumberPerPage.pc,
        };
      }

      case 'sp': {
        return {
          currentPageNum: currentPageNumber,
          startIndex: 0,
          endIndex: currentPageNumber * maxNumberPerPage.sp,
        };
      }
    }
  }),
);

export function LogoListPager() {
  // ここで無限レンダリングが発生
  const pager = useAtomValue(
    pagerState({ displayType: 'pc', contentsId: 'logo', maxNumberPerPage: 9 }),
  );

  return <Pager pager={pager} />;
}
```

この事象は[React Dev Tools](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=ja)を使用することで確認することができます。

## **無限レンダリングが発生した原因**

この問題の原因は、atomFamily の引数に直接オブジェクトを渡していたことにあります。

まず、Recoil の atomFamily では、引数オブジェクトをシリアライズして文字列として比較する仕組みになっています。そのため、内容が同じオブジェクトであれば参照が異なっていても同じ atom として扱われ、キャッシュされた atom を正しく取得できます。

一方、Jotai の atomFamily は、引数として渡された値の参照をもとに atom を識別し、内部的にキャッシュします。オブジェクトの場合、たとえ内容が同じでも参照が異なれば別の値と認識されます。そのため、レンダリングのたびに新しいオブジェクトが生成されると、同じパラメータであっても毎回異なる atom が作成されてしまいます。

結果として、useAtomValue などのフックが参照する atom が毎回異なるものとなり、レンダリングが無限に発生してしまいました。

```typescript
// Recoil（内部でシリアライズ比較されるため、キャッシュされているatomが再利用される）
useRecoilValue(atomFamily({ value: 1 }));

// Jotai（レンダリングの度に参照が変わるため、毎回新しいatomが生成される）
useAtomValue(atomFamily({ value: 1 }));
```

Recoilでシリアライズ比較をしている箇所
<https://github.com/facebookexperimental/Recoil/blob/c1b97f3a0117cad76cbc6ab3cb06d89a9ce717af/packages/recoil/recoil_values/Recoil_atomFamily.js#L137>

Jotaiで参照比較をしている箇所（Map.prototype.get()では === の比較、つまりparamがオブジェクトの場合、参照の同一性を比較する）
<https://github.com/pmndrs/jotai/blob/53bbf932b06a6d64f79d80ed1ff5f2ca6f401e4c/src/vanilla/utils/atomFamily.ts#L42>

この atomFamily の実装上の違いを十分に把握しないまま Recoil から Jotai へ移行を進めてしまった結果、予期せず無限レンダリングが発生してしまいました。

## **修正方法**

今回の問題は、コンポーネントのレンダリングごとに毎回新しいオブジェクトリテラルを作成し、そのオブジェクトを atomFamily の引数として渡していたことにあります。

React ではコンポーネントが再レンダリングされるたびに、関数本体も再実行されます。そのため、コンポーネント内部でオブジェクトリテラルを定義すると、再レンダリングのたびに新しいオブジェクト（異なる参照）が生成されます。

この問題を解消するためには、パラメータオブジェクトの参照が毎回変わらないようにする必要があります。そこで、パラメータとなるオブジェクトをコンポーネントの外（ファイルのトップレベル）で一度だけ定義し、それを atomFamily の引数として渡すように修正しました。これにより、コンポーネントが何度レンダリングされても params は同じ参照のまま維持され、不要な atom の再生成や無限レンダリングが発生しなくなりました。

```typescript

export function LogoListPager() {
  const pager = useAtomValue(pagerState(params));

  return <Pager pager={pager} />;
}
```

同様に、useState などから動的に生成した値を含むパラメータオブジェクトの場合も、useMemo でオブジェクトをメモ化することで、レンダリングごとに同じ参照が使われるようにしました。これにより、参照等価性が保たれ、不要な再レンダリングを防ぐことができます。

```typescript
export function LogoListPager() {
  const [displayType, setDisplayType] = useState('pc');
  const params = useMemo(() => ({
    displayType,
    contentsId: 'logo',
    maxNumberPerPage: 9
  }, [displayType]);
  const pager = useAtomValue(pagerState(params));

  return <Pager pager={pager} />;
}
```

これにより無限レンダリングの事象を解消することができました。

## **atomFamilyのareEqualオプションによる比較方法のカスタマイズ**

Jotai v2以降では、atomFamily の第2引数に比較用の関数（ areEqual ）を渡すことで、atom の一意性を判定する際のロジックを柔軟にカスタマイズできます。

公式ドキュメントでは、[**fast-deep-equal**](https://github.com/epoberezkin/fast-deep-equal)を利用することで Recoil の atomFamily や selectorFamily と同じような値の等価性による比較が再現できる例が掲載されています。

```typescript
// 引用： https://jotai.org/docs/utilities/family#atomfamily
import { atom } from 'jotai'
import { atomFamily } from 'jotai/utils'
import deepEqual from 'fast-deep-equal'

const fooFamily = atomFamily((param) => atom(param), deepEqual)
```

しかし、fast-deep-equal は約5年間新しいリリースがなく、保守状況に不安があったことから、今回はこの方法の採用は見送りました。そのため、今回紹介した通り「参照の同一性」による比較で atom を管理する方法を選択しています。

## おわりに

今回は、Recoil から Jotai への移行に伴い、atomFamily の引数の扱い方の違いによって無限レンダリングが発生した事例をご紹介しました。

状態管理ライブラリを比較する際、使い方やAPIの違いに注目しがちですが、キャッシュ管理の仕組みについては普段意識する機会が少なく、今回のような問題につながることがあります。今後は内部実装の仕組みにも目を向けながら、移行作業に取り組んでいきたいと思います。
