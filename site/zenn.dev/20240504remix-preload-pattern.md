---
https://zenn.dev/apple_yagi/articles/1b5dd876b3c5e5
---

# Remix で preload パターンを実装する

**Author:** やなぎ (Yanagi)
**Published:** 2024/05/04
**Updated:** 2024/05/05

## Overview

This article demonstrates two methods of implementing preload patterns in Remix:

### 1. Using `defer`

The first approach uses Remix's `defer` method to create streaming/delayed responses. Here's a basic implementation:

```typescript
export const loader = async () => {
  const aStillRunningPromise = loadSlowDataAsync();

  const isAuthenticated = await getIsAuthenticated();
  if (!isAuthenticated) {
    redirect('/login');
  }

  return defer({
    slowPromise: aStillRunningPromise,
  });
}

export default function Page() {
  const { slowPromise } = useLoaderData<typeof loader>();

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Await resolve={slowPromise}>
        {(resolved) => <p>{resolved.title}</p>}
      </Await>
    </Suspense>
  );
}
```

### 2. Using React's `use` API

The second approach leverages React's `use` API for a cleaner implementation:

```typescript
export default function Page() {
  const { slowPromise } = useLoaderData<typeof loader>();
  const resolved = use(slowPromise);

  return (
    <p>{resolved.title}</p>
  );
}
```

This method eliminates the need for `<Suspense>` and `<Await>` wrappers, resulting in more streamlined HTML.

The article notes this is a Remix version inspired by a previous Next.js preload pattern implementation.
