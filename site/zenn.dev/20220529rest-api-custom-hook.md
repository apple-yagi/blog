---
https://zenn.dev/apple_yagi/articles/bc52d83ec61aee
---

# REST APIと良い感じに通信するHookを自作する

**Author:** やなぎ
**Published:** 2022/05/29

## Overview

This article discusses creating a custom React hook for making API calls with improved reusability. The author presents three approaches to handling API communication in React components:

### 1. Bad Approach: Direct API Communication in Component

- API logic mixed directly within the component
- Difficult to reuse
- Complex for multiple API interactions

### 2. No Bad Approach: Separating Logic into a Specific Hook

- Extracts API logic into a dedicated hook
- Improves component readability
- Still creates multiple similar hooks for different endpoints

### 3. Good Approach: Generic Fetch Hook

- Uses TypeScript generics to create a flexible `useFetch` hook
- Accepts URL and data type as parameters
- Enables easy, reusable API communication across components

## Key Code Example: Generic Fetch Hook

```typescript
export const useFetch = <T>(url: string) => {
  const [data, setData] = useState<T>();
  const [isLoading, setLoading] = useState(true);
  const [isError, setError] = useState(false);

  useEffect(() => {
    (async () => {
      try {
        const res = await fetch(url);
        const data = await res.json();
        setData(data);
      } catch (err) {
        console.error(err);
        setError(true);
      } finally {
        setLoading(false);
      }
    })();
  }, []);

  return { data, isLoading, isError };
};
```

## Conclusion

The author suggests moving beyond traditional hook separation by creating more generic, reusable hooks that can simplify API interactions across React applications.
