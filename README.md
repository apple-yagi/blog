# blog

apple-yagi's blog repo

## How to convert website to markdown

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest $site_url > $output_markdown_file_name
# example
docker run --rm -i markitdown:latest https://developers.prtimes.jp/2024/06/06/replace-top-page-with-nextjs/ > output.md
```

## Linting

- Install dependencies once via `pnpm install`.
- Run `pnpm lint` to execute both markdownlint and textlint. The textlint pass loads `@textlint-ja/textlint-rule-preset-ai-writing` alongside the JA technical-writing preset to keep prose consistent.
- Use `pnpm lint:fix` for auto-fixes when possible.
