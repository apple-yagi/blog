---
https://developers.prtimes.jp/2024/12/11/improving-performance-with-tanstack-virtual/
---

# TanStack Virtualを使用してパフォーマンス改善をした話



こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

先日、メディアリスト機能のリニューアルを実施しました。リニューアルの詳細については、以下のプレスリリースをご覧ください。

株式会社PR TIMESのプレスリリース（2024年11月12日 15時30分）新着メディア表示機能とメディア検索UI刷新。PR TIMESメディアリスト機能をリニューアル

本エントリーでは、私が開発を担当したインポートリスト機能にTanStack Virtualを導入して、リリース前にパフォーマンス改善を行なった話についてご紹介します。

目次

## インポートリスト機能とは

インポートリスト機能は、企業が取引のあるメディアの情報をPR TIMESのメディアリストに簡単に取り込むことができる機能です。

リニューアル以前はページネーションを利用してメディアを表示していましたが、新しいバージョンではページネーションを廃止し、全メディアを直接表示するように変更しました。更に、リスト内でメディアを検索する機能が追加されました。そして、以前はメディアごとに追加、編集、削除操作を個別に行う必要があったのに対し、リニューアルを通じて一括でこれらの操作が可能となりました。

## **パフォーマンスの課題**

メディアの数が多いリストでは、膨大なデータが1ページに表示されるため、重くなったり動作が不安定になったりする問題がリリース前に確認されました。

インポートリスト機能にはメディアの数に上限が設けられていないため、実際に5,000件以上のメディア情報を含むリストが存在し、この問題がリリース前に解決を必要としていました。

## パフォーマンス改善案

上記の問題は、1ページ内に表示される大量のメディア（コンポーネント）が原因でした。そこで以下の3つの解決策を検討しました。

1. 無限スクロール導入し、必要なデータを都度バックエンドから取得
2. UIをページネーションに戻す
3. 仮想スクロールを導入する

無限スクロールの案は、リスト内のメディアを検索する際に、DBに保存前のメディアも一緒に検索できるようにする必要があったため、不採用にしました。ページネーションに戻す案については、インポートリスト機能のリニューアルで実現したいユーザー体験が損なわれてしまうため、不採用にしました。結果として、現状のUIを保ったまま、パフォーマンスの改善が見込める仮想スクロールを実装する案を採用しました。

## 実装について

仮想スクロールの実装には [TanStack Virtual](https://tanstack.com/virtual/latest) を使用しました。他の選択肢として [react-window](https://github.com/bvaughn/react-window) がありましたが、動的に高さが変わるリストとの相性が悪い問題があったり、2018年からバージョンの更新が止まっていることから採用を見送りました。

GitHub

TanStack VirtualはVueやSolid、Svelteなどでも使用できますが、PR TIMESではReactを使用しているため、 `@tanstack/react-virtual` で実装を行なっています。

実装コードは以下の通りです。

```
import {useVirtualizer} from '@tanstack/react-virtual';

type Props = {
  readonly mediaOutlets: MediaOutlet[];
}

export function OurMediaOutletTable({
  mediaOutlets
}: Props) {
  const ref = useRef<HTMLElement>(null);

  const virtualizer = useVirtualizer({
    count: mediaOutlets.length,
    // スクロールする要素を取得する
    // スクロールしない要素を設定するとうまく動かなくなる
    getScrollElement: () => ref.current,
    estimateSize: () => 90,
    overscan: 3,
  });

  return (
    <section ref={ref} className={styles.section}>
      <table className={styles.table}>
        <OurMediaOutletTableHead />
        <tbody
          style={{
            height: `${virtualizer.getTotalSize()}px`,
            position: 'relative',
          }}
          className={styles.tbody}
        >
          {virtualizer.getVirtualItems().map((virtualItem) => (
            <OurMediaOutletTableRow
              key={virtualItem.key}
              ref={virtualizer.measureElement}
              // コンポーネント内で `data-index={dataIndex}` をしている
              // TanStack Virtualは data-index の値を見て、
              // アイテムの高さを動的に取得しているため、これを忘れるとうまく動かなくなる
              dataIndex={virtualItem.index}
              mediaOutlet={mediaOutlets[virtualItem.index]}
              style={{
                position: 'absolute',
                transform: `translateY(${virtualItem.start}px)`,
              }}
            />
          ))}
        </tbody>
      </table>
    </section>
  );
}
```

上記の実装により、5万件以上のメディアが含まれるリストを表示してもパフォーマンスの問題は解消されました。

## まとめ

PR TIMESでは昨年からリニューアルプロジェクトを進めており、UIの刷新を行なっています。その際にパフォーマンス面に注意して実装しないといけない箇所がいくつもありますが、リニューアルを通して実現したいユーザー体験を維持しつつも、パフォーマンスを向上していきたいと考えています。今回の事例もその1つですが、今後のリニューアルプロジェクトでも継続していきたいです。

## **We are hiring!**

フロントエンドエンジニアを含む各種ポジションでの採用を進めています！興味があればぜひご応募ください。

株式会社PR TIMES

株式会社PR TIMESでは現在03-4. 開発部 フロントエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
