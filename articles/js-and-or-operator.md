---
title: "【JavaScript】AND 演算子と OR 演算子についてまとめてみた"
emoji: "🥯"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "TypeScript", "React"]
published: true
date: "2025.02.22"
---

# JavaScript の AND 演算子と OR 演算子について

JavaScript には論理演算を行うための演算子として、AND 演算子（`&&`）と OR 演算子（`||`）があります。これらは条件分岐や値の短絡評価などで頻繁に使われます。

:::message
備忘録として様々な演算子の中から昨今の開発で使う頻度が多くなった演算子について連続で記事を書いてきましたが、今回 AND 演算子と OR 演算子をまとめ、これを一旦最終回としたいと思います。
過去の記事はこちら：
[【JavaScript】三項演算子についてまとめてみた](https://zenn.dev/hayatech/articles/js-ternary-operator)
[【JavaScript】null 合体演算子についてまとめてみた](https://zenn.dev/hayatech/articles/js-nullish-operator)
:::

## AND 演算子（論理積 `&&`）

### 基本的な使い方

AND 演算子（`&&`）は、**両方の条件が真（true）の場合にのみ真（true）を返す**演算子です。

```javascript
console.log(true && true); // true
console.log(true && false); // false
console.log(false && true); // false
console.log(false && false); // false
```

### 実用的な例

以下のように、複数の条件を組み合わせることで、特定の条件を満たしたときだけ処理を実行できます。

```javascript
const age = 25;
const hasLicense = true;

if (age >= 18 && hasLicense) {
  console.log("運転できます！");
} else {
  console.log("運転できません。");
}
```

このコードでは、`age`が 18 以上で、かつ`hasLicense`が`true`のときのみ「運転できます！」と表示されます。

### JSX での活用

React では`&&`を使って条件付きレンダリングを行うことができます。

```jsx
const isLoggedIn = true;

function WelcomeMessage() {
  return <div>{isLoggedIn && <p>ようこそ！ログインに成功しました！</p>}</div>;
}
```

この例では、`isLoggedIn`が`true`のときのみ、メッセージが表示されます。

## OR 演算子（論理和 `||`）

### 基本的な使い方

OR 演算子（`||`）は、**少なくとも 1 つの条件が真（true）であれば、結果が真（true）になる**演算子です。

```javascript
console.log(true || true); // true
console.log(true || false); // true
console.log(false || true); // true
console.log(false || false); // false
```

### 実用的な例

以下のように、デフォルト値を設定するのに使われることが多いです。

```javascript
const username = "";
const displayName = username || "ゲスト";

console.log(displayName); // "ゲスト"
```

このコードでは、`username`が空文字（falsy な値）の場合、`displayName`には「ゲスト」が代入されます。

### JSX での活用

React では、`||`を使ってデフォルト値を設定することができます。

```jsx
const user = null;

function UserGreeting() {
  return <p>{user || "ユーザーが見つかりません"}</p>;
}
```

この例では、`user`が`null`または`undefined`のとき、「ユーザーが見つかりません」と表示されます。

## まとめ

| 演算子     | 説明                               |
| ---------- | ---------------------------------- |
| AND 演算子 | 両方が`true`のときのみ`true`を返す |
| OR 演算子  | どちらかが`true`なら`true`を返す   |

AND 演算子（`&&`）と OR 演算子（`||`）は、条件分岐やデフォルト値の設定など、さまざまな場面で活用されます。
特に React では、条件付きレンダリングやデフォルトの表示内容を設定する際に頻繁に使うので覚えておくと役に立ちます！
