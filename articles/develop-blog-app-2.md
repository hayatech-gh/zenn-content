---
title: "ZENN の記事を高速で反映させるブログサイトを開発🧑🏼‍💻 〜開発編〜"
emoji: "🧑🏼‍💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs", "TypeScript", "TailwindCss"]
published: true
date: "2025.05.18"
---

# ZENN の記事を高速で反映させるブログサイトを開発 🧑🏼‍💻（Next.js × TypeScript × Tailwind CSS 使用、Vercel で無料運用） 〜開発編〜

## はじめに

仕事やプライベートで Next.js や TypeScript、TailwindCSS などを習得した知見を活かしたいと思いましたので、それらを活用した個人開発の一環として、「HayaTech-Blog」というブログサイトを作りました。ZENN に投稿している記事を取得して、エンジニア向けのブログとして公開しています。

**HayaTech-Blog**
URL：[https://hayatech-blog.vercel.app/](https://hayatech-blog.vercel.app/)

このブログサイトは、ZENN に投稿した Markdown 形式の記事を GitHub リポジトリで管理し、それを GitHub API 経由で取得・表示する仕組みがあります。Vercel を利用することで、無料でデプロイできるだけでなく、高速かつ安定した運用を実現しています。

設計編では、ブログサイトの設計思想と主要な技術要素について解説しました。本稿の開発編では、ブログサイトの主要な部分のコード実装に焦点を当てて解説します。

## コード

主要な機能を実現するためのコードについて、そのポイントを解説していきます。
可読性を重視するため、型定義や細部の実装は省略している場合があります。予めご了承ください 🙇‍♂️。

## Markdown 記事の取得・加工のコード解説

### 1\. GitHub 上の Markdown ファイル一覧を取得する

```ts:lib/utility/getarticle.ts
export async function fetchGithubRepo(url: string) {
  try {
    const res = await fetch(url, {
      headers: { Authorization: `token ${process.env.GITHUB_TOKEN}` },
    });
    if (!res.ok) throw `ステータスコードエラー：${res.status}`;
    return res.json();
  } catch (err) {
    console.log(`GitHubリポジトリのデータを取得でエラー：${err}`);
  }
}
```

この関数では、Next.js の `fetch` API を使用して GitHub API にリクエストを送信し、指定された URL のリポジトリコンテンツ（ここでは ZENN 記事が格納されたディレクトリ）の情報を JSON 形式で取得します。`.env`ファイルにある環境変数 `GITHUB_TOKEN` を使用して認証を行うことで、プライベートリポジトリへのアクセスも可能になります。エラーハンドリングとして、レスポンスが OK でない場合はエラーをスローし、取得中に例外が発生した場合はコンソールにエラーログを出力します。

---

### 2\. 各記事の本文とメタデータを取得・解析する

```ts:lib/utility/getArticle.ts
export async function fetchGithubMakeArticle(url: string, fileName: string) {
  try {
    const res = await fetch(url + fileName, {
      headers: { Authorization: `token ${process.env.GITHUB_TOKEN}` },
    });
    if (!res.ok) throw `ステータスコードエラー：${res.status}`;

    const data = await res.json();
    const buffer = Buffer.from(data.content, 'base64');
    const fileContents = buffer.toString('utf-8');
    const matterResult = matter(fileContents);

    if (!matterResult.data.published) return;

    return {
      id: fileName.replace(/\.md$/, ''),
      ...(matterResult.data as ArticleMeta),
      content: matterResult.content,
      from: 'Zenn',
    };
  } catch (err) {
    console.log(`contentfetchデータの処理中にエラー：${err}`);
  }
}
```

この関数は、指定されたファイル名に対応する個別の Markdown ファイルの内容を GitHub API から取得し、解析します。取得したコンテンツは Base64 エンコードされているため、`Buffer.from(data.content, 'base64').toString('utf-8')` でデコードします。次に、`gray-matter` ライブラリを使用して Markdown ファイルを解析し、フロントマター（記事のメタデータ）と本文を分離します。`published: false` の記事は除外され、記事の ID、メタデータ、本文、そして取得元（'Zenn'）を含むオブジェクトを返します。

---

### 3\. 記事一覧をまとめて取得して整形する

```ts:lib/mdData.ts
export async function getMdsData(): Promise<Article[]> {
  const zennArticles: ArticleResponse[] = await fetchGithubRepo(
    'https://api.github.com/repos/hayatech-gh/zenn-content/contents/articles',
  );

  const datas = await Promise.all(
    zennArticles.map(async (article) => {
      return await fetchGithubMakeArticle(
        'https://api.github.com/repos/hayatech-gh/zenn-content/contents/articles/',
        article.name,
      );
    })
  );

  return datas.filter(Boolean) as Article[];
}
```

`getMdsData` 関数は、GitHub リポジトリの記事一覧を取得した後、`Promise.all` を使用して各記事のデータを並列に取得し、整形します。`zennArticles.map` で各記事ファイルに対して `fetchGithubMakeArticle` を呼び出し、その結果を Promise の配列として受け取ります。`Promise.all` が完了すると、未公開の記事（`fetchGithubMakeArticle` で `return` されなかった要素）を `filter(Boolean)` で除外し、最終的に `Article[]` 型の整形された記事データの配列を返します。

---

### 4\. Markdown を HTML に変換し、Zenn 拡張記法に対応する

```ts:lib/mdData.ts
export async function getHtmlContent(article: Article) {
  const unifiedContent = await unified()
    .use(remarkParse)
    .use(remarkMath)
    .use(remarkDirective)
    .use(customDirectives)
    .use(remarkGfm)
    .use(remarkRehype, { allowDangerousHtml: true })
    .use(rehypeRaw)
    .use(rehypeKatex)
    .use(rehypeHighlight)
    .use(rehypeStringify)
    .process(transformZennMd(article.content));

  return {
    ...article,
    content: unifiedContent.toString(),
  };
}
```

この関数では、`unified` を利用して、Markdown を HTML に変換します。`remarkParse` で Markdown を抽象構文木 (AST) に変換し、`remarkMath`, `remarkDirective`, `customDirectives`, `remarkGfm` などのプラグインで数式、カスタムディレクティブ、GitHub Flavored Markdown などの拡張記法に対応します。`transformZennMd` 関数では、Zenn 独自の記法（`:::message` ブロックや画像サイズ指定など）を変換するためのカスタム処理を行います。

---

## トップページ (`app/page.tsx`) のコード解説

### 1\. Markdown データの取得とソート

```tsx:app/page.tsx
const allMdsData = await getMdsData();
const sortedMdData = await getSortedMdsData(allMdsData);
```

トップページでは、まず `getMdsData()` 関数を呼び出して GitHub から全ての公開済み Markdown 記事データを取得します。その後、`getSortedMdsData()` 関数を用いて、取得した記事データを公開日時の降順にソートします。これにより、常に最新の記事がトップに表示されるようになります。

### 2\. ページネーション処理の実装

```tsx:app/page.tsx
const currentPage = params.page
  ? parseInt(Array.isArray(params.page) ? params.page[0] : params.page, 10)
  : 1;
const paginatedBlogs = sortedMdData.slice(
  (currentPage - 1) * pageLimit,
  currentPage * pageLimit
);
```

Next.js の `searchParams` を利用して、URL クエリパラメータから現在のページ番号 (`page`) を取得します。もし `page` パラメータが存在しない場合は、デフォルトで最初のページ (`1`) を表示します。`pageLimit` で定義された一件あたりの表示件数に基づいて、`slice` メソッドを使用してソート済みの記事データから現在のページに表示する記事の配列 (`paginatedBlogs`) を抽出します。

### 3\. 記事カードの描画

```tsx:app/page.tsx
{
  paginatedBlogs.map(({ id, title, emoji, date, topics, type }) => (
    <li key={id}>
      <Link href={`/blogs/${id}`}>
        <div className={`${type === "idea" ? "bg-idea" : "bg-tech"}`}>
          <div>{emoji}</div>
        </div>
        <h2>{title}</h2>
        <span>{type == "tech" ? "Tech" : "Idea"}</span>
        <Topics topicList={topics} />
        <Date dateString={date} />
      </Link>
    </li>
  ));
}
```

`paginatedBlogs` 配列を `map` 関数でループ処理し、各記事に対応する UI（記事カード）を生成します。記事の `type` プロパティに基づいて、Tailwind CSS のカスタムクラス (`bg-idea`, `bg-tech`) を動的に適用し、カテゴリごとの背景色を切り替えています。記事のタイトル、絵文字、カテゴリ、トピック一覧、日付などのメタデータをそれぞれのコンポーネントで表示し、`<Link>` コンポーネントで記事詳細ページへの遷移を設定しています。

---

## 詳細ページ (`app/blogs/[id]/page.tsx`) のコード解説

### 1\. 記事データの取得と Markdown から HTML への変換

```tsx:app/blogs/[id]/page.tsx
const allBlogsData = await getMdsData();
const blogData = getMdData(allBlogsData, id);
const convertedBlogData = await getHtmlContent(blogData);
```

詳細ページでは、まず `getMdsData()` で全ての記事データを取得し、`getMdData()` 関数を使って、ルーティングパラメータ (`id`) に対応する特定の記事データを取り出します。その後、`getHtmlContent()` 関数を呼び出し、記事の Markdown コンテンツを HTML 文字列に変換します。この HTML 文字列は、後で `dangerouslySetInnerHTML` を使用して安全に DOM に挿入されます。

### 2\. 記事内容の HTML 表示

```tsx:app/blogs/[id]/page.tsx
<div className="md-html">
  <div dangerouslySetInnerHTML={{ __html: convertedBlogData.content }} />
</div>
```

変換された HTML コンテンツを Web ページに表示するために、React の `dangerouslySetInnerHTML` プロパティを使用します。これにより、Markdown で記述された記事の内容がレンダリングされます。

### 3\. コメント・いいね機能の埋め込み

```tsx:app/blogs/[id]/page.tsx
<Like blogId={id} />
<Comment blogId={id} />
<Board blogId={id} />
```

記事詳細ページには、ユーザーがリアクションやコメントを行えるように、`<Like />`, `<Comment />`, `<Board />` といったカスタムコンポーネントを配置しています。これらのコンポーネントは、それぞれの記事の `id` を `blogId` プロパティとして受け取り、Supabase などのバックエンドサービスと連携して、いいねのカウント表示やコメントの投稿・表示機能を提供します。

## まとめ

以上の開発編では、個人で開発したブログサイトの主要な部分のコード実装に焦点を当てて解説しました。
次回、運用編では、Vercel を利用した効率的なデプロイと運用、 SEO 対策について掘り下げて解説していく予定です！
