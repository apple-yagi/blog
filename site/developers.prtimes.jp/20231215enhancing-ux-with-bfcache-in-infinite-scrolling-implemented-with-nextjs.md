---
https://developers.prtimes.jp/2023/12/15/enhancing-ux-with-bfcache-in-infinite-scrolling-implemented-with-nextjs/
---

# BFCacheを利用してNext.jsで実装した無限スクロールのUX改善をした話

**Author:** 柳 龍哉  
**Date:** 2023年12月15日

## Next.jsに移行した初期の実装

初期の実装では、`getServerSideProps`で検索結果の1ページ目を取得し、そのデータを Tanstack Query に hydrate する形で実装しました。

主な目的は2つありました：

1. 検索結果が0件の場合にHTTPステータスコード404を返すこと
2. コンテンツの表示を速くすること

サンプルコード:

```javascript
export const getServerSideProps = async ({req, res, query}) => {
  const {search_word: searchWord} = query;

  const queryClient = new QueryClient();
  const searchResultResponse = await getSearchResult({searchWord, page: 1});

  await queryClient.prefetchInfiniteQuery(
    ['searchResult', searchWord],
    () => searchResultResponse.data,
  );

  if (searchResultResponse.data.total === 0) {
    res.statusCode = 404;
  }

  return {
    props: {
      searchWord,
      dehydratedState: dehydrate(queryClient)
    }
  }
}
```

## 課題：ページ遷移後にブラウザバックすると、2ページ目以降のデータが消える問題

この問題は無限スクロールのUIでよく見られる課題で、ユーザーからは「毎回別タブで開く」という不満の声がありました。

具体的な問題：

- ユーザーが無限スクロールで複数ページを表示
- 詳細ページに遷移
- ブラウザバックで戻ると、1ページ目のデータのみが表示される

## BFCacheを利用したデータの復元

BFCache（Back/Forward Cache）を利用して、アプリケーションコードにほとんど手を加えることなくデータを復元しました。

### 実施した2つの対応

1. **BFCacheの有効化確認**
   - Chrome DevToolsでBFCacheが正常に動作していることを確認
   - Next.jsのデフォルト設定でBFCacheは有効

2. **ページビューイベントの調整**
   - `pageshow`イベントでBFCacheからの復帰を検知
   - Google Analyticsの重複計測を防ぐ設定を追加

### 実装例

```javascript
useEffect(() => {
  const handlePageShow = (event) => {
    // BFCacheからの復帰時の処理
    if (event.persisted) {
      // 必要に応じてデータの再取得や状態の復元
      console.log('Page restored from BFCache');
    }
  };

  window.addEventListener('pageshow', handlePageShow);
  
  return () => {
    window.removeEventListener('pageshow', handlePageShow);
  };
}, []);
```

## 結果

BFCacheの活用により：

1. **UX改善**: ユーザーがブラウザバック時に以前のスクロール位置とデータが保持される
2. **開発コストの削減**: アプリケーションコード側での複雑な状態管理が不要
3. **パフォーマンス向上**: ページの再読み込みが不要で、瞬時にページが表示される

## まとめ

BFCacheは無限スクロールのUX問題を解決する強力なソリューションです。Next.jsでは特別な設定なしに利用でき、ユーザーエクスペリエンスの大幅な改善を少ない実装コストで実現できました。

無限スクロールを実装する際は、BFCacheの活用を検討することをお勧めします。
