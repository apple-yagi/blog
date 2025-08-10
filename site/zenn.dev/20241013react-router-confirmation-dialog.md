---
https://zenn.dev/apple_yagi/articles/958c9f39815b16
---

# React Routerで画面遷移前に破棄確認ダイアログを表示する

**Author:** やなぎ
**Published:** 2024/10/13

## Overview

This article explains how to implement a confirmation dialog in React Router before navigating away from a page with unsaved changes, using React Router version 6.27.0.

## Key Code Example

```jsx
import { useCallback, useState } from "react";
import { unstable_usePrompt, useBeforeUnload } from "react-router-dom";

const message = "行った変更が保存されない可能性があります。";

export function Form() {
  const [name, setName] = useState("");
  const isDirty = name !== "";

  // Handle page reload or non-SPA navigation
  useBeforeUnload(
    useCallback(
      (event) => {
        if (!isDirty) return;

        if (window.confirm(message) === false) {
          event.preventDefault();
          event.returnValue = "";
        }
      },
      [isDirty]
    )
  );

  // Handle SPA navigation
  unstable_usePrompt({
    message,
    when: ({ currentLocation, nextLocation }) =>
      isDirty && currentLocation.pathname !== nextLocation.pathname,
  });

  return (
    <form>
      <label>
        Name:
        <input
          onChange={(event) => setName(event.target.value)}
          value={name}
          type="text"
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Key Insights

- Uses two hooks: `useBeforeUnload` and `unstable_usePrompt`
- `useBeforeUnload` handles page reload and non-SPA navigation
- `unstable_usePrompt` handles SPA navigation
- Requires using `createBrowserRouter` for these hooks to work

## Caveats

The React Router documentation recommends considering alternative approaches, such as:

- Persisting unsaved state in `sessionStorage`
