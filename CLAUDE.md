# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a blog repository containing technical articles in Markdown format. Articles are organized under two main directories:
- `site/developers.prtimes.jp/` - Development blog articles from PR TIMES
- `site/zenn.dev/` - Technical articles from Zenn

## Commands

### Linting

```bash
# Lint all Markdown files (runs both markdownlint and textlint in parallel)
pnpm lint

# Lint Markdown structure/formatting only
pnpm lint:md

# Lint Japanese text quality only
pnpm lint:text

# Auto-fix lint issues
pnpm lint:fix
```

### Converting Websites to Markdown

To convert web articles to Markdown format using the markitdown Docker container:

```bash
# Build markitdown container (one-time setup)
git clone git@github.com:microsoft/markitdown.git
cd markitdown
docker build -t markitdown:latest .

# Convert a webpage to Markdown
docker run --rm -i markitdown:latest $site_url > $output_markdown_file_name
```

## Text Quality Configuration

The repository uses two text linters with specific configurations:

- **textlint**: Configured for Japanese technical writing with AI writing presets. Applied to articles in `site/**/*.md`
- **markdownlint**: Configured with relaxed rules allowing inline HTML, flexible header styles, and no line length restrictions

## Package Management

This project uses pnpm v10.14.0 as the package manager. Install dependencies with `pnpm install`.