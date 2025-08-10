---
https://zenn.dev/apple_yagi/articles/fde1bdfafee65f
---

# Tailwind Variants: Observations and Challenges

**Author:** やなぎ (Yanagi)
**Published:** June 7, 2023
**Updated:** July 2, 2023

## Key Challenges with Tailwind Variants

### 1. VS Code Tailwind CSS IntelliSense Issue

- IntelliSense doesn't work by default with Tailwind Variants
- Solution: Add the following to `settings.json`:

```json
{
  "tailwindCSS.experimental.classRegex": [
    ["tv\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"]
  ]
}
```

### 2. Prettier Plugin Tailwind CSS Compatibility

- Prettier plugin doesn't work with Tailwind Variants
- Solution: Update `prettier.config.js`:

```javascript
module.exports = {
  plugins: [require("prettier-plugin-tailwindcss")],
  tailwindFunctions: ["tv"],
};
```

### 3. VariantProps Type Issue (Resolved)

- Initially, `VariantProps` was not functioning correctly with TypeScript 5.1+
- The issue has since been resolved in a later update

## Conclusion

Tailwind Variants looks promising, with some initial configuration challenges that have mostly been addressed.
