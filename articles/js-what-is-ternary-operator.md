---
title: "【JavaScript】三項演算子についてまとめてみた"
emoji: "☘️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "React"]
published: true
date: "2025.01.28"
---

# JavaScript の三項演算子について

私は普段の仕事では条件分岐の際に if 文の方が使用頻度が高いですが、プライベートでは三項演算子を使って条件分岐させるのも少なくないです。
今回は JavaScript の**三項演算子**について、その基本的な使い方をおさらいします。この記事では、三項演算子の概念や実際のコード例、React の JSX での使い方に加えて、if 文との比較についても解説します。

## 三項演算子とは

三項演算子は、JavaScript の条件分岐で便利に使えるシンプルな演算子です。たった 1 行のコードで条件分岐を書くことができます。

### 構文

```
条件 ? 条件がtrueの場合 : 条件がfalseの場合
```

試しに一つの例を見てみましょう。

### 三項演算子で書く場合

以下は、API からのレスポンスが成功した場合にデータを表示し、失敗した場合にエラーメッセージを表示するコード例です。

```javascript
const apiResponse = {
  success: true,
  data: "ユーザーデータ",
};

const result = apiResponse.success ? apiResponse.data : "エラーが発生しました";

console.log(result); // "ユーザーデータ"
```

この例では、apiResponse.success が true ならば data を返し、false ならばエラーメッセージを返します。三項演算子を使用すると、条件分岐を簡潔に記述できます。

### if 文との比較

三項演算子を使わずに if 文で同じロジックを記述すると、次のようになります。

```javascript
const apiResponse = {
  success: true,
  data: "ユーザーデータ",
};

let result;

if (apiResponse.success) {
  result = apiResponse.data;
} else {
  result = "エラーが発生しました";
}

console.log(result); // "ユーザーデータ"
```

if 文を使う場合、条件ごとに処理を分ける必要があるため、記述が少し長くなります。

## JSX 内で三項演算子を使う

特に、三項演算子は React の JSX で使いやすいです。

### JSX の例

以下は、isLoading の状態に応じてコンポーネントの内容を切り替える例です。
（なお、React を使う際は、通常 useState を用いて状態を管理しますが、この例では簡潔さを重視しています。）

```jsx
const LoadingIndicator = ({ isLoading }) => {
  return (
    <div>
      {isLoading ? <p>ローディング中...</p> : <p>内容が読み込まれました!</p>}
    </div>
  );
};

// 例の使用
<LoadingIndicator isLoading={true} />;
```

この例では、isLoading が true の場合に「ローディング中...」が表示され、false の場合に「内容が読み込まれました!」が表示されます。

### if 文との比較

一応、JSX 内では if 文を使うことも可能ですが、直接的に条件式を記述できないため、場合によっては関数の外で処理を分ける必要があります。

```jsx
const LoadingIndicator = ({ isLoading }) => {
  let content;

  if (isLoading) {
    content = <p>ローディング中...</p>;
  } else {
    content = <p>内容が読み込まれました!</p>;
  }

  return <div>{content}</div>;
};
```

このように、if 文を使う場合は若干記述が増えるため、簡潔な条件分岐には三項演算子が適しています。

## 三項演算子の注意点

三項演算子は簡潔に条件分岐を記述できる反面、条件が複雑になると可読性が低下します。例えば、入れ子の三項演算子を使う場合には注意が必要です。

```javascript
const result = isError
  ? "エラーが発生しました"
  : isLoading
  ? "読み込み中です"
  : "成功しました";
```

こうした条件が複雑な際には、if 文の方が可読性が良くなります。

```javascript
let result;
if (isError) {
  result = "エラーが発生しました";
} else if (isLoading) {
  result = "読み込み中です";
} else {
  result = "成功しました";
}
```

## まとめ

三項演算子は、if 文よりも簡潔に条件分岐を書きたい場合に役立ちます。特に、React の JSX でよく使われる場面があります。ただし、複雑な条件を三項演算子で書くとコードが読みづらくなる場合もあるため、適切な場面で使い分けることが大切です。
