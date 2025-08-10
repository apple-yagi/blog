---
https://zenn.dev/apple_yagi/articles/61e59cb1c56221
---

# Next.jsで使える機能を制限するESLintプラグイン（eslint-plugin-no-next-restricted）を作ってみた

**Author:** やなぎ (apple_yagi)
**Published:** 2025/03/31

## Overview

The author created an ESLint plugin called `eslint-plugin-no-next-restricted` to limit the usage of certain Next.js features. The plugin provides two main lint rules:

1. Prevent importing `next/image`
2. Prevent importing `next/link`

## Motivation

The plugin was created for scenarios where:

- Image optimization services like Fastly IO or imgix are already in use, making `next/image` redundant
- Server-Side Rendering pages are cached on CDN
- Only specific pages use Next.js features

## Key Implementation Points

### Testing Approach

- Used `node --experimental-strip-types --test` for unit testing
- Avoided using Vitest or Jest
- Referenced testing strategies from the `css-modules-kit` ESLint plugin

### TypeScript Configuration

- Enabled `rewriteRelativeImportExtensions` in `tsconfig.json`
- Noted potential bugs with this configuration

## Conclusion

The plugin aims to provide more control over Next.js dependencies and improve future maintainability by restricting specific framework features.
