---
https://zenn.dev/apple_yagi/articles/9e1d7dd94c3d47
---

# App RouterでEmotionを使う時の致命的な辛み

**Author:** やなぎ (apple_yagi)
**Published:** 2024/09/21

## Overview

The article discusses challenges encountered when using Emotion with Next.js App Router, highlighting two primary issues:

### 1. Metadata Definition Problem

When using Emotion in App Router, defining page-specific metadata becomes problematic. The key challenges include:

- Emotion requires all files with HTML tags to be Client Components (`"use client"`)
- Metadata can only be defined in Server Components
- This creates a conflict where you cannot set unique metadata for each page

### 2. Fetch Warning Issue

In App Router, fetching data directly in components can trigger warnings, especially in Client Components. The warning suggests:

> "Components suspended by uncached promises. Creating promises in client components or hooks is not yet supported."

## Conclusion

The author notes these issues were discovered while casually attempting to use Emotion with App Router and invites readers to share potential workarounds.

## Key Takeaways

- Emotion and App Router integration has significant compatibility challenges
- Metadata and data fetching become complex with the current implementation
- Developers may need alternative strategies when combining these technologies
