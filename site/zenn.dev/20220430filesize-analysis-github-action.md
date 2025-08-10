---
https://zenn.dev/apple_yagi/articles/c568c8f0061e82
---

# filesize-analysisというGithub Actionを作った話

**Author:** やなぎ ([@apple-yagi](https://twitter.com/apple_yagi))
**Published:** 2022/04/30

## Overview

The article introduces `filesize-analysis`, a GitHub Action that:

- Retrieves file sizes for specified file types in a repository
- Adds comments to pull requests with file size information
- Works across different frontend build tools

## Key Concepts

### Minimal GitHub Actions Configuration

The workflow can be configured with just a few lines:

```yaml
- name: Analysis
  uses: apple-yagi/filesize-analysis@v1
  with:
    out_dir: "./dist"
    ext: "bundle.js|chunk.js"
    github_token: ${{secrets.GITHUB_TOKEN}}
```

### Tool-Agnostic Approach

The action is designed to be independent of specific frontend build tools like webpack, esbuild, or vite. It extracts file sizes based on user-specified directories and file extensions.

## Motivation

The author created this action to:

- Easily track file sizes in pull requests
- Avoid manually checking bundle sizes
- Provide a lightweight solution for monitoring frontend application sizes

## Conclusion

While similar tools exist, the author believes `filesize-analysis` offers unique flexibility by supporting multiple languages and build systems.
