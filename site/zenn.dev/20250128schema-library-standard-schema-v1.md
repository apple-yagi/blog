---
https://zenn.dev/apple_yagi/articles/362d830bdb80f4
---

# スキーマライブラリの共通インターフェイスを提供するStandard Schema v1をご紹介

**Author:** やなぎ (apple_yagi)
**Published:** 2025/01/28

## Overview

The article introduces Standard Schema v1, a new approach to creating a common interface for validation libraries. Key points include:

- Existing validation libraries like Zod, Valibot, and Arktype require specific adapters when integrating with form libraries
- Standard Schema aims to create a "common interface" for validation libraries
- The goal is to make switching between validation libraries easier

## Key Highlights

### Problem with Current Validation Libraries

"Currently, validation libraries like Zod, Valibot, and Arktype require specific adapters when integrating with libraries like TanStack Form."

### Standard Schema Solution

"Standard Schema defines a common interface for validation libraries, allowing libraries to implement this standard and create compatibility between different validation tools."

### Benefits

- Eliminates the need for library-specific adapters
- Increases flexibility in library combinations
- Makes switching validation libraries easier

## Example Usage

```typescript
// Simple validation using Standard Schema
const userSchema = z.object({
  name: z.string(),
});

export function Form() {
  const form = useForm({
    validators: {
      onChange: userSchema,
    },
    // ... other form configuration
  });
}
```

## Conclusion

The introduction of Standard Schema simplifies validation library integration and promises more flexibility in future development.
