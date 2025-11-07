---
https://zenn.dev/apple_yagi/articles/e3131cb5bb6b8f
---

# React Hook FormのstandardSchemaResolverを使えばzod v4に対応できるよ

**Author:** やなぎ (apple_yagi)  
**Date:** 2025/05/25

## はじめに

今年の2月に[React Hook Form](https://react-hook-form.com/)は[Standard Schema](https://github.com/standard-schema/standard-schema)に対応しました。Standard Schemaの導入により、React Hook Formは特定のバリデーションライブラリに依存せず、標準化されたインターフェースを通して柔軟に連携できるようになりました。

## zod v4がReact Hook Formで動かない問題

React Hook FormとZodを組み合わせて使う際には、`zodResolver`を使うのが一般的ですが、残念ながらこのzodResolverはまだzod v4に対応していません。

例えば次のコードでは型エラーが発生します。

```typescript
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod/v4';

const formSchema = z.object({
  name: z.string(),
});

export function Form() {
  const {register} = useForm({
    resolver: zodResolver(formSchema),
    defaultValues: {
      name: '',
    },
  });

  return <input {...register('name')} />;
}
```

## 解決策：standardSchemaResolver を使おう

このzod v4との互換性の問題に対しては、`standardSchemaResolver`を使用することで解決できます。以下のように書き換えればOKです。

```typescript
import { standardSchemaResolver } from '@hookform/resolvers/standard-schema';
import { useForm } from 'react-hook-form';
import { z } from 'zod/v4';

const formSchema = z.object({
  name: z.string(),
});

export function Form() {
  const { register } = useForm({
    resolver: standardSchemaResolver(formSchema),
    defaultValues: {
      name: '',
    },
  });

  return <input {...register('name')} />;
}
```

## メリット

- zodの内部実装に依存しない
- React Hook Formのアップデートを待たずにzod v4への移行が可能
- より標準化されたバリデーションインターフェースを提供

このアプローチにより、ライブラリの破壊的変更に対してより堅牢なコードを書くことができます。
