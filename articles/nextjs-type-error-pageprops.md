---
title: "【Next.js(v15)】エラー解決法：Type '***' does not satisfy the constraint"
emoji: "🙀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs", "TypeScript"]
published: true
date: "2025.03.10"
---

# 【Next.js 15】エラー解決法：Type error: Type '\*\*\*' does not satisfy the constraint

Next.js(v15)及び TypeScript でアプリを開発しているのですが、ビルドする際に「Type error: Type '\*\*\*' does not satisfy the constraint」というエラーによく遭遇しました。同じようなエラーが出てしまった方向けに、解決方法を解説します。

## Next.js 15 とは？

Next.js は、React をベースとした Web アプリケーションフレームワークです。サーバーサイドレンダリング（SSR）や静的サイト生成（SSG）など、高度な機能を手軽に実装できるため、多くの開発者に利用されています。

Next.js 15 では、パフォーマンスの向上や開発体験の改善など、さまざまなアップデートが行われました。その中でも特に注目すべきは、App Router 機能(app ディレクトリ) の安定版リリースです。これにより、より柔軟で効率的な開発が可能になりました。

## エラー：Type error: Type '\*\*\*' does not satisfy the constraint

今回解説するエラーは、Next.js 15 で TypeScript を使用している際に、型の制約を満たさない場合に発生します。具体的には、`app`ディレクトリ内の`page.tsx`や`layout.tsx`などで、props の型定義が正しく行われていない場合に発生することが多いです。

## エラー解決例 1：searchParams の型定義

まずは、`searchParams`の型定義に関するエラー解決例についてです。

### エラーが発生するコード

```typescript
// app/page.tsx
type HomeProps = {
  searchParams?: { [key: string]: string | string[] | undefined };
};

export default async function Home({ searchParams }: HomeProps) {
  const params = searchParams ?? {};
  // ...
}
```

上記のコードでは、`searchParams`の型定義が`Promise`でラップされていないため、エラーが発生します。Next.js 15 では、`searchParams`は`Promise`でラップされた非同期の値として扱われます。

### 正しいコード

```typescript
// app/page.tsx
type HomeProps = {
  searchParams?: Promise<{ [key: string]: string | string[] | undefined }>;
};

export default async function Home({ searchParams }: HomeProps) {
  const params = (await searchParams) ?? {};
  // ...
}
```

上記のコードでは、`searchParams`の型定義を`Promise<{ [key: string]: string | string[] | undefined }>`に変更し、`await`を使用して非同期処理の結果を取得しています。

### 解説

- `searchParams`は、URL のクエリパラメータを表すオブジェクトです。
- Next.js 15 では、`searchParams`は非同期の値として扱われるため、`Promise`でラップする必要があります。
- `await`を使用することで、`Promise`が解決されるまで処理を一時停止し、解決された値を取得できます。
- `??`は、null 合体演算子と呼ばれ、左辺の値が`null`または`undefined`の場合に右辺の値を返します。参考：[【JavaScript】null 合体演算子についてまとめてみた](https://zenn.dev/hayatech/articles/js-nullish-operator)

## エラー解決例 2：params の型定義

次に、`params`の型定義に関するエラー解決例を見ていきましょう。

### エラーが発生するコード

```typescript
// app/blogs/[id]/page.tsx
type BlogProps = {
  params: { id: string };
};

export default async function Blog({ params }: BlogProps) {
  const { id } = params;
  // ...
}
```

上記のコードでは、`params`の型定義が`Promise`でラップされていないため、エラーが発生します。`searchParams`と同様に、`params`も Next.js 15 では非同期の値として扱われます。

### 正しいコード

```typescript
// app/blogs/[id]/page.tsx
type BlogProps = {
  params: Promise<{ id: string }>;
};

export default async function Blog({ params }: BlogProps) {
  const { id } = await params;
  // ...
}
```

上記のコードでは、`params`の型定義を`Promise<{ id: string }>`に変更し、`await`を使用して非同期処理の結果を取得しています。

### 解説

- `params`は、動的なルートパラメータを表すオブジェクトです。
- `searchParams`と同様に、`params`も非同期の値として扱われるため、`Promise`でラップする必要があります。
- `await`を使用することで、`Promise`が解決されるまで処理を一時停止し、解決された値を取得できます。

## エラー解決のポイント

- Next.js 15 では、`searchParams`や`params`などの props は非同期の値として扱われるため、`Promise`でラップする必要があります。
- `await`を使用して、非同期処理の結果を取得する必要があります。
- 型定義を正しく行うことで、TypeScript による型チェックを有効活用し、開発効率を向上させることができます。

## まとめ

本記事では、Next.js 15 で「Type error: Type '\*\*\*' does not satisfy the constraint」というエラーが発生した場合の解決方法を解説しました。
Next.js 15 では、App Router の導入により、より柔軟で効率的な開発が可能になりましたが、新しい機能や概念を理解する必要があるため、とても大変です 💦
本記事が、Next.js 15 での開発におけるエラー解決の一助となれば幸いです。
