---
title: "ZENN の記事を高速で反映させるブログサイトを開発🧑🏽‍💻 〜運用編〜"
emoji: "🧑🏽‍💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# ZENN の記事を高速で反映させるブログサイトを開発 🧑🏼‍💻（Next.js × TypeScript × Tailwind CSS 使用、Vercel で無料運用） 〜運用編〜

## はじめに

仕事やプライベートで Next.js や TypeScript、TailwindCSS などを習得した知見を活かしたいと思いましたので、それらを活用した個人開発の一環として、「HayaTech-Blog」というブログサイトを作りました。ZENN に投稿している記事を取得して、エンジニア向けのブログとして公開しています。

**HayaTech-Blog**
URL：[https://hayatech-blog.vercel.app/](https://hayatech-blog.vercel.app/)

このブログサイトは、ZENN に投稿した Markdown 形式の記事を GitHub リポジトリで管理し、それを GitHub API 経由で取得・表示する仕組みがあります。Vercel を利用することで、無料でデプロイできるだけでなく、高速かつ安定した運用を実現しています。

設計編では、ブログサイトの設計思想と主要な技術要素について、開発編では、ブログサイトの主要な部分のコード実装に焦点を当てて解説しました。最終稿となる運用編では、このブログサイトを支える Vercel の機能、実施した SEO 対策、そして今後の展望について詳しく解説していきます。

## Vercel

・Vercel の説明
Vercel の説明

・Analytics と SpeedInsights についての説明
ここに画像が入ります。（Analytics ツールの外観）
ここに Analytics の説明が入ります。
ここに画像が入ります。（SpeedInsights ツールの外観）
ここに SpeedInsights の説明が入ります。

・注意
設計段階では広告の導入も検討していたが、Vercel の利用規約により、広告の掲載や収益化サービスの利用不可でしたので断念しました

## SEO 対策

・セマンティックな構築
セマンティックな HTML（JSX）構造を意識し、コードを書きました。
また、Markdown 形式の記事が自動で HTML に変換されることから、記事を書く際には構造に気をつけています。

・Google SEO
Google Search Console に登録
これにより、Google や Yahoo の検索エンジンでサイト名を入力すると、上位に表示されるようになりました。
ここに画像が入ります。（検索エンジンで上位の様子）

・Bing SEO
Bing Webmaster Tools に登録
これにより、Bing の検索エンジンでサイト名を入力すると、上位に表示されるようになりました。
ここに画像が入ります。（検索エンジンで上位の様子）

## 今後の展望

以下のアップデートを考えています。
可能な限り SSR・SSG を活かしたサイトにしたいが、以下の機能を導入したらクライアント側になる可能性もあるので、十分に検討したい。
共通
・検索機能
トップページ
・フィルター
・並び替え機能
Blog ページ
・目次機能

## まとめ

運用編では、〜〜〜について解説しました。
開発にあたり、Next.js や TypeScript などの知見が高まりました。
