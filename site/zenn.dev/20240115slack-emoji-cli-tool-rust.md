---
https://zenn.dev/apple_yagi/articles/2b2d5b3052c206
---

# Slack用の絵文字をCLIで生成できるツールをRustで作ってみた

**Author:** やなぎ
**Date:** 2024/01/15
**Topics:** Rust, Slack, Tech

## Overview

The author created a CLI tool in Rust for generating Slack emojis quickly. The tool, called `egc`, allows users to generate custom emojis directly from the command line.

## Features

- Generate Slack emojis with a simple command
- Customize text color (e.g., `egc 完全に理解した -c yellow`)
- Easy installation via Homebrew

### Installation

```bash
brew tap apple-yagi/tap
brew install egc
```

## Technical Details

### Libraries Used

- clap
- image
- imageproc
- rusttype

## Challenges

The author encountered an issue with text color rendering, specifically with intermediate RGB colors like orange, where text overlaps become darker.

## Demonstration

Example command:

```bash
egc 完全に理解した
```

This generates an emoji image similar in size to existing emoji generators.

## Repository

GitHub: [https://github.com/apple-yagi/egc](https://github.com/apple-yagi/egc)

The author notes that most of the code was generated with GitHub Copilot.
