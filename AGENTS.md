# Repository Guidelines

## Project Structure & Content Sources

The repository tracks Markdown exports of posts from developers.prtimes.jp and Zenn. Articles live under `site/developers.prtimes.jp` and `site/zenn.dev`, each file named `<yyyymmdd><slug>.md` to match publication order. Supporting docs such as `README.md` describe conversion tooling, and `CLAUDE.md` holds agent notes. No compiled assets are stored; treat the repo as content-first.

## Build, Test, and Development Commands

Install dependencies with `pnpm install` before running any lint task. `pnpm lint:md` runs markdownlint-cli2 across the workspace, ensuring heading levels and spacing. `pnpm lint:text` applies textlint with Japanese technical-writing presets to the article corpus. Use `pnpm lint` to run all lint jobs in parallel, and `pnpm lint:fix` to auto-correct formatting issues.

## Coding Style & Naming Conventions

Write articles in Markdown with a three-dash YAML front matter block that only lists the canonical URL. Begin each file with an H1 article title, followed by bold `Author` and `Published` metadata lines. Use fenced code blocks with language hints (for example, start a block with three backticks and `jsx`). Keep file names lowercase, date-prefixed, and hyphenated, matching the upstream slug (e.g., `20241013react-router-confirmation-dialog.md`). Prefer English body copy unless quoting Japanese UI text.

## Testing Guidelines

Linting is the primary quality gate. Run `pnpm lint` locally before opening a pull request, and ensure the textlint rules pass; failing rules usually point to style or terminology issues. There are no automated unit tests, but proofread for accuracy and broken links.

## Commit & Pull Request Guidelines

Existing history favors concise, descriptive commit messages in Japanese that state the source and action. Continue that style or use present-tense English equivalents, keeping prefixes such as `zenn.dev` when updating specific feeds. Pull requests should summarize the article set touched, note any manual edits beyond the automated export, and link to the original posts. Attach screenshots only when visual formatting changed.

## Content Conversion Workflow

To import a new article, follow the Docker-based `markitdown` steps in `README.md`, saving the output into the appropriate site folder. Review the generated Markdown for heading hierarchy and confirm embedded URLs resolve.
