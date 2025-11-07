---
https://developers.prtimes.jp/2025/05/09/migrate-from-emotion-to-css-modules/
---

# EmotionからCSS Modulesに移行しました

こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

PR TIMESのフロントエンドではこれまで[Emotion](https://github.com/emotion-js/emotion)を使ってスタイリングを行っていましたが、2024年6月からCSS Modulesへの移行作業を進めており、先日その作業がすべて完了しました。本エントリーでは、移行の背景や技術選定、移行作業中に発生した問題などについてご紹介します。

目次

## 移行の背景

社内の一部機能を2024年3月頃に[Remix SPA Mode](https://remix.run/docs/en/main/guides/spa-mode)へ移行しようとしましたが、Emotionが動作しなかったため断念し、その経験からCSS Modulesへの移行を検討し始めました。

また、今後React Server Components（RSC）が主流になる可能性があることを想定すると、このままEmotionを使い続けるのはリスクだと感じるようになりました。Emotionによって開発者体験は確かに向上していましたが、スタイリングライブラリのせいでフレームワークの進化に追従しづらくなったり、他のフレームワークへの移行が困難になったりすることは避けたいと考え、移行に踏み切りました。

## 移行先の技術選定

Emotionの移行先として、CSS Modulesと[TailwindCSS](https://tailwindcss.com/)の2つを候補に挙げていました。ADR（Architectural Decision Record）にCSS Modulesを選んだ理由を書いていたため、その一部を抜粋して紹介します。

> **Proposal / 提案 (required)**
>
> CSS Modulesに移行することを提案します。 理由は以下の通りです。
>
> * WebpackやViteなどのバンドラーで自動でバンドルされる
>   * Remix SPAやNext.jsのApp Routerで設定をいじらずにそのまま使える → フレームワークの乗り換えが楽になる
> * パフォーマンスが良い
>   * build時にCSSファイルが出力されるため、JSでスタイルの計算をする必要がなくなる
>
> また、CSS Modulesをそのまま使用すると、Emotionと比べて開発者体験が著しく低下してしまうため、[happy-css-modules](https://github.com/mizdra/happy-css-modules)もセットで使用するつもりです。
>
> CSS Modules以外の移行先候補としてTailwindCSSがあります。
>
> TailwindCSSはCSS Modulesと同様に、フレームワークの乗り換えが楽になり、パフォーマンスも良くなるというメリットがあります。しかし、以下の点でデメリットもあります。
>
> * TailwindCSSのバージョンを更新する手間が発生する
>   * メジャーバージョンアップなどでBreaking Changeなどが発生した際に、記法を変える必要やtailwind.confingを書き換える必要がある
>
> * classNameが長くなってしまうデメリットがある（人による）
>   * CSSプロパティのcontentにbase64が直接指定されている箇所があり、TailwindCSSにそのまま転記するのが困難
>   * Tailwind Variantsを導入することで一定は可読性を良くすることは可能ではある <https://zenn.dev/apple_yagi/articles/fde1bdfafee65f>
>
> 上記の理由の中でも、スタイルを書くということに対して、バージョンアップなどのメンテナンスコストをあまり発生させたくないため、TailwindCSSを採用したくないというのが一番大きいところです。
>
> そして、CSS Modulesにも今後何かしらのBreaking Changeが発生する可能性があるかもしれません。そのため、本来であればPlainなCSSを採用したいところではあります。
>
> PlainなCSSでも [@scope](https://developer.mozilla.org/ja/docs/Web/CSS/%40scope) を使えば、スコープありなスタイルを書くことが可能です（まだ実用段階ではないですが）。他にも @layer、 @container などもあります。
>
> 今後のCSSの進化を考えると、スタイリングライブラリなどが必要ない世界が訪れると思っています。そのためにも、PlainなCSSに限りなく近いCSS Modulesを採用したいと思っています。

TailwindCSSを選ばなかった理由は、メジャーバージョンアップによるメンテナンスコストを懸念していたためです。実際、TailwindCSS v4のリリースでtailwind.config.jsがCSSファイルに変更されるなどのBreaking Changeが発生し、当時（2024年4月頃）懸念していたことが現実になりました。

また、候補にゼロランタイムのCSS-in-JSが上がらなかった理由としては、デファクトスタンダードなライブラリがまだ存在せず、多くのライブラリが毎月のように登場していたため、不安定だと感じたことがあります。さらに、メンテナンスコストが高く、フレームワークごとにアドホックな対応が必要になりそうだったことも理由の一つです。

## CSS Modulesへの移行

CSS Modulesを導入するにあたり使用したライブラリと、移行作業中に発生した問題について紹介します。

### happy-css-modulesの導入

CSS Modulesの型定義ファイルを生成するために、happy-css-modulesを利用しています。こちらについては昨年投稿したエントリーで紹介しているため、そちらをご参照ください。

あわせて読みたい

### postcss-nestingの導入

EmotionのTagged Template Literalでは、以下のように擬似要素をスタイリングできます。実際にPR TIMESでもこの方法を用いて擬似要素をスタイリングしていました。

```
import { css } from "@emotion/react";

const button = css`
  color: black;

  &:hover {
    color: red;
  }
`;

/*
出力されるCSSのイメージ

.button {
  color: black;
}

.button:hover {
  color: red;
}
*/

```

このCSS記法（CSS Nesting）はプレーンなCSSでも可能ですが、Safariではバージョン16.5以降でしかサポートされていません。PR TIMESではバージョン15以降をサポート対象としているため、CSS Nestingをそのまま利用することはできません。しかし、移行作業をなるべくコピー＆ペーストで行いたかったため、この問題を解決するために[postcss-nesting](https://github.com/csstools/postcss-plugins/tree/main/plugins/postcss-nesting#readme)を導入し、CSS Nestingの記法をそのまま記述できるようにしました。

しかし、Emotionでは以下のように「&」がなくても「&」がある場合と同じ挙動をします。そのため、単純にコピー＆ペーストすると、Emotionの時とCSSの当たり方が異なる場合がありました。

```
import { css } from "@emotion/react";

const button = css`
  color: black;

  // & がない
  :hover {
    color: red;
  }
`;

/*
出力されるCSSのイメージ

.button {
  color: black;
}

.button:hover {
  color: red;
}
*/
```

具体的には、Emotionでは擬似要素の前に「&」があってもなくても、親要素に対する擬似要素にCSSが適用されます。しかし、プレーンなCSSでは「&」がない場合、子孫要素の擬似要素にCSSが適用されるという違いがあります。

```
/** 擬似要素の前に&がある時 */
.button {
  color: black;

  &:hover {
    color: red;
  }
}

.button {
  color: black;
}

.button:hover {
  color: red;
}

/** 擬似要素の前に&がない時 */
.button {
  color: black;

  :hover {
    color: red;
  }
}

.button {
  color: black;
}

/** 子孫要素の:hoverに対してスタイリングされる */
.button :hover {
  color: red;
}
```

そのため、移行作業では「&」を付けることを常に意識しました。

### JavaScriptの変数をCSS Modulesで使用する

Emotion では以下のように JavaScript の変数を直接スタイルに適用できます。

```
function Button() {
  // ランダムにフォントサイズ(12 ~ 16px)を決定する
  const randomFontSize = Math.floor(Math.random() * 5) + 12;

  return (
    <button
      type='button'
      css={css`
        font-size: ${randomFontSize}px;
      `}
    >
      ボタン
    </button>
  );
}
```

一方、CSS ModulesではJavaScriptの変数を直接CSSプロパティで使用できないため、JavaScriptの変数をCSS変数に割り当てる形で移行しました。

```
import styles from "./button.module.css";

function Button() {
  // ランダムにフォントサイズ(12 ~ 16px)を決定する
  const randomFontSize = Math.floor(Math.random() * 5) + 12;

  return (
    <button
      type='button'
      style={
        {
          '--font-size': randomFontSize,
        } as CSSProperties
      }
      className={styles.button}
    >
      ボタン
    </button>
  );
}

/*
button.module.css

.button {
  font-size: var(--font-size)px;
}
*/
```

また、propsの値によって適用するCSSが全く異なるケースでは、data属性を利用して移行しています。

```
import {css} from '@emotion/react';

type Props = {
  readonly variant: 'raised' | 'flat' | 'outlined' | 'text';
};

function Button({variant}: Props) {
  return (
    <button type='button' css={[button, variants[variant]]}>
      ボタン
    </button>
  );
}

const button = css`
  font-size: 16px;
  padding: 10px 20px;
`;

const variants = {
  raised: css`
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
  `,
  flat: css`
    background-color: transparent;
    color: #007bff;
    border: none;
  `,
  outlined: css`
    background-color: transparent;
    color: #007bff;
    border: 2px solid #007bff;
    border-radius: 4px;
  `,
  text: css`
    background-color: transparent;
    color: #007bff;
    border: none;
  `,
} as const;
```

```
import styles from "./button.module.css";

type Props = {
  readonly variant: 'raised' | 'flat' | 'outlined' | 'text';
};

function Button({variant}: Props) {
  return (
    <button type='button' className={styles.button} data-variant={variant}>
      ボタン
    </button>
  );
}

/*
button.module.css

.button {
  font-size: 16px;
  padding: 10px 20px;

  &[data-variant='raised'] {
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
  }

  &[data-variant='flat'] {
    background-color: transparent;
    color: #007bff;
    border: none;
  }

  &[data-variant='outlined'] {
    background-color: transparent;
    color: #007bff;
    border: 2px solid #007bff;
    border-radius: 4px;
  }

  &[data-variant='text'] {
    background-color: transparent;
    color: #007bff;
    border: none;
  }
}
*/
```

### CSSの詳細度の問題

CSS Modulesに移行すると、Emotionではあまり気にする必要がなかったCSSの詳細度を意識する必要が出てきました。

たとえば、親コンポーネントから子コンポーネントのスタイルを上書きする場合、Emotionでは必ず親コンポーネントから渡したスタイルが優先されます。

```
import { css } from "@emotion/react";

function Parent() {
  return (
    <Children
      css={css`
        font-size: 12px;
      `}
    />
  );
}

function Children(props: ComponentPropsWithoutRef<'p'>) {
  return (
    <p
      css={css`
        // propsで渡されたcssが適用されるため、font-sizeは12pxになる
        font-size: 16px;
      `}
    >
      Children
    </p>
  );
}
```

しかし、CSS Modulesの場合は、ビルドされたCSSファイル中のclassNameの順番によって詳細度が変化します。そのため、親コンポーネントからclassNameを渡しても、ビルド時のCSSファイルの出力結果により、スタイルが適用されることもあれば、適用されないこともあります。

```
import styles from "./parent.module.css";
import clsx from "clsx";

export function Parent() {
  return (
    <Children className={styles.parent} />
  );
}

  return (
    <p
      className={clsx(styles.children, className)}
    >
      Children
    </p>
  );
}

/*
parent.module.css

.parent {
  font-size: 12px;
}

.children {
  font-size: 16px;
}
*/
```

```
.parent__GvGjD {
  font-size: 12px;
}

/**
 * CSSは一番最後に記述されたスタイルが適用されるため、
 * この場合親コンポーネントから渡したスタイルではなく子コンポーネントのスタイルが適用される.
 */
.children__YA2mH {
  font-size: 16px;
}
```

ビルド時に出力される className の順序を厳密に制御するのは難しいため、同じ CSS プロパティを上書きしたい場合は inline style を使用するように対応しました。inline style は詳細度が最も高いため、CSS の出力順に関係なくスタイルを上書き可能です。

```
export function Parent() {
  return (
    {/** classNameではなくinline styleを使用するように変更 */}
    <Children style={{ fontSize: 12 }} />
  );
}
```

ただし、inline styleはmedia queryが使用できません。そのため、一部の箇所では親コンポーネントからclassNameを渡し、 `!important` を使ってスタイル調整を行っています。

この問題の根本的な解決策としては、親コンポーネントから子コンポーネントのCSSプロパティを直接上書きするのではなく、コンポーネントを分けるか、propsの値によってCSSプロパティを変化させる設計が望ましいと考えています。

### EmotionとCSS Modulesが混在した時の問題

CSS Modules への移行はコンポーネント単位で少しずつ進めました。その結果、1 つの HTML 要素に CSS Modules と Emotion の両方でスタイルを適用するケースも発生しました。両者で同じ CSS プロパティを扱うと、css props の渡し方によって適用される CSS が変わりました。

まず、以下のようにcss propsとclassNameが設定されている場合は、css propsの方が適用されます。

```
import styles from "./text.module.css";

const textCss = css`
  color: red;
`;

function Text() {
  return (
    {/** css propsで指定しているcolorが適用される */}
    <p css={textCss} className={styles.text}>
      テスト
    </p>
  );
}

/*
text.module.css

.text {
  color: blue;
}
*/
```

しかし、スプレッド構文を使用して親コンポーネントから受け取ったcss propsをHTMLタグに適用した場合、classNameを代入する前と後で挙動が変わります。

```
import styles from "./children.module.css";

const childrenCss = css`
  color: red;
`;

function Parent() {
  return (
    <Children css={childrenCss} />
  );
}

function Children(props: Props) {
  return (
    <div>
      {/** classNameの前でスプレッド構文を展開するとclassNameのcolorが適用される */}
        テスト
      </p>
      {/** classNameの後にスプレッド構文を展開するとcss propsのcolorが適用される */}
        テスト
      </p>
    </div>
  );
}

/*
children.module.css

.text {
  color: blue;
}
*/
```

また、classNameの前でスプレッド構文を展開すると、そもそもEmotionが生成するclassNameが上書きされてしまうため、Emotionで記述したスタイル自体が適用されなくなります。逆にclassNameの後にスプレッド構文を展開した場合、classNameに代入していた値（上記の例の styles.text ）が上書きされて消え、EmotionのclassNameのみが残ります。以下の画像は実際に上記のコンポーネントをブラウザで表示した際のHTMLです（ `class="_text_1he59_1"` はCSS ModulesのclassNameで、 `class="css-rnnx2x"` はEmotionのclassNameです）。

PR TIMESでは親コンポーネントからcss propsを渡す際にスプレッド構文を多用していたため、inline styleを使う方法に変更したり、スプレッド構文を使わずにcss propsを明示的に渡す方法に変えながら移行を進めました。

## まとめ

PR TIMESはメンテナンスコスト、フレームワークの進化の追従・乗り換えのしやすさを重視し、EmotionからCSS Modulesに移行しました。この移行にはフロントエンドに興味のあるバックエンドエンジニアや、インターンなどフロントエンドメンバー以外の協力が多くあり、移行し切ることができました。今後はReact 19へのバージョンアップのためにRecoilの依存を取り除くという大きな課題もあるため、引き続き取り組んでいきます。

## We are hiring!

フロントエンドエンジニアを含む各種ポジションでの採用を進めています！興味があればぜひご応募ください。

株式会社PR TIMES

株式会社PR TIMESでは現在03-4. 開発部 フロントエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
