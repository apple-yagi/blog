# 指定したURLの記事をMarkdownに変換し、記事のタイトルと本文のみを残すプロンプト

1. 既存のMarkdownファイルを確認し、URLを取得
2. markitdownを使用してURLをMarkdownに変換
3. 変換されたMarkdownから記事に関係ない箇所（ナビゲーション、サイドバー、採用情報、関連記事など）を削除
4. 記事の先頭にURLを挿入
5. タイトル（#見出し）と本文のみを残す

具体的なコマンド例：

## URLを取得してMarkdownに変換

URL=$(grep -o 'https://[^>]*'/path/to/existing/file.md | head -1) docker run --rm -i markitdown:latest "$URL" > temp.md

## 記事のタイトルと本文のみを抽出し、URLを先頭に追加
  
cat > /path/to/output/file.md << EOF
$URL

$(sed -n '/^# /,/^## おわりに/p'
temp.md | sed '/^## We are
hiring!/,$d' | sed '/^##
この記事を書いた人/,$d' | sed '/^##
関連記事/,$d' | sed '/^\[.*\](.*)/d' |
sed '/^カテゴリ/,/^# /d' | sed
'/^!\[.*\](.*)/d' | sed '/^\*
\[.*\]/d')
EOF

## 不要なファイルを削除

rm temp.md

または、より効率的に一つのコマンドで実行する場合：

URL="<https://developers.prtimes.jp/202>
5/07/31/recoil-to-jotai-atomfamily-inf
inite-rendering/"
OUTPUT_FILE="/Users/ryuya.yanagi/works
pace/oss/blog/site/developers.prtimes.
jp/20250731recoil-to-jotai-atomfamily-
infinite-rendering.md"

docker run --rm -i markitdown:latest
"$URL" | \
awk 'BEGIN{print "'"$URL"'"; print ""}
 /^# /{found=1} found &&
!/^カテゴリ|^\[|^!\[|^\* \[|^##.*この
記事を書いた人|^##.*関連記事|^##.*We
are hiring/
{if(!/^##.*おわりに/){print} else
{print; exit}}' > "$OUTPUT_FILE"

## 不要な文言を削除

以下の形式の文言を削除する

```
2025
2/26
```

```
URLをコピーしました！
```

「...」で省略されている文章

## markdownlintを実行し、エラーを修正する
