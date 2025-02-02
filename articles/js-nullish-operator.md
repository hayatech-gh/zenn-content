---
title: "【JavaScript】null 合体演算子についてまとめてみた"
emoji: "🈚️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "TypeScript", "React"]
published: false
date: "2025.02.02"
---

# JavaScript の null 合体演算子について

前回は様々な演算子の中で[三項演算子](https://zenn.dev/hayatech/articles/js-ternary-operator)を取り上げましたが、今回は **null 合体演算子** についてまとめてみたいと思います。

## null 合体演算子とは

`??` の 左側の値が `null` または `undefined` のときに、右側の値を返す 演算子です。
(もし null と undefined が分からない場合は参考記事をご覧ください。参考記事：[JavaScript での undefined と null の違い](https://zenn.dev/fujee/articles/a3eec3709fe7ac)）

### 構文

```javascript
値1 ?? 値2;
```

`値1` が `null` または `undefined` の場合に `値2` を返します。

### シンプルな例

```javascript
const name = null;
const displayName = name ?? "Zenn";
console.log(displayName); // "Zenn"
```

この場合、`name` は `null` なので、右側の `"Zenn"` が返されます。

### 注意：論理和`||` との違い

`||` は Falsy な値 (`0`, `''`, `false` など) も「偽」とみなすため、意図しない動作になることがあります。一方で `??` は **null または undefined の場合のみ** 右側の値を返します。

（論理和||と論理積&&については後で記事を書こうと思います。）

## 実用例

### API レスポンスの処理

```javascript
const apiResponse = {
  items: undefined,
};

const items = apiResponse.items ?? [];
console.log(items); // [] (空の配列)
```

API からのデータが `undefined` でも、空の配列 `[]` を設定することでエラーを防げます。

## まとめ

null 合体演算子`??` は **`null` または `undefined` の場合のみ** 右側の値を返します。
設定値のデフォルト処理や、API レスポンスの処理などで`null` や`undefined`が発生する可能性がある際に使うと、エラーを回避できるので有効です。
