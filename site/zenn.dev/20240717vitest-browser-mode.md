---
https://zenn.dev/apple_yagi/articles/b2b96af1fd355c
---

# Vitest Browser Mode がアツい

**Author:** やなぎ (apple_yagi)
**Published:** 2024/07/17

## Background

The article discusses the challenges of component testing with Vitest, traditionally using jsdom or happy-dom to simulate browser environments. These approaches have limitations:

> これまでVitestでコンポーネントのテストを行う時は、jsdom や happy-dom を使ってブラウザ環境を偽装していました。

The author highlights that mocking browser environments often requires complex setup and doesn't provide a truly realistic testing scenario.

## Key Benefits of Vitest Browser Mode

1. Real browser environment testing
2. No need for complex mocking
3. Support for component tests without heavy frameworks
4. Easy setup

## Demo and Example

The article provides a detailed example using a Radix UI Tooltip component, demonstrating how Browser Mode simplifies testing compared to jsdom approaches.

### Performance Comparison

Test Environment | Execution Speed
--------------- | ---------------
Browser Mode    | 95ms
jsdom           | 75ms

## Conclusion

The author concludes that Vitest Browser Mode offers a more straightforward and realistic testing approach, particularly for complex components requiring native browser APIs.

> Browser Modeを使用することにより、テスト環境の構築（jsdomの設定など）が不要となり、リアルな環境でコンポーネントのテストをすることができるようになりました。
