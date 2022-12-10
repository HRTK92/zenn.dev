---
title: "Next.jsとtailwindcss、nextauth、prismaを使用した掲示板"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "nextjs", "tailwindcss", "nextauth", "prisma"]
published: true
---

https://github.com/HRTK92/next-boards

このサイトは、Next.jsとtailwindcss、nextauth、prismaを使用して作成した掲示板です。

### 技術的な特徴

- Next.js: サーバーサイドレンダリングを提供するJavaScriptフレームワーク
- tailwindcss: CSSフレームワーク
- nextauth: Next.js用の認証ライブラリ
- prisma: データベースを扱うためのORMツール

### 主な機能

- ユーザー登録
- ログイン
- 名前の変更
- スレッドの作成
  - パブリックスレッド
  - プライベートスレッド
  - タグ付け

### 使い方

1. リポジトリをクローンする

```bash
git clone https://github.com/HRTK92/next-boards.git
```

2. パッケージをインストールする

```bash
yarn install
```

3. 環境変数を設定する

```env:.env
DATABASE_URL={DATABASE_URL}
SECRET={NEXTAUTH_SECRET}

LINE_CLIENT_ID=
LINE_CLIENT_SECRET=

NEXTAUTH_URL=

NEXT_PUBLIC_SITE_NAME=
```

`DATABASE_URL`は、MongoDBのURLを指定します。
`SECRET`は、NextAuthのシークレットキーを指定します。
`LINE_CLIENT_ID`と`LINE_CLIENT_SECRET`は、LINEログインのクライアントIDとクライアントシークレットを指定します。
`NEXTAUTH_URL`は、NextAuthのURLを指定します。
`NEXT_PUBLIC_SITE_NAME`は、サイト名を指定します。

:::message
`prisma/schema.prisma`の`provider`を変更することで、データベースを変更できます。
:::

4. 開発サーバーを起動する

```bash
yarn dev
```

![](<https://raw.githubusercontent.com/HRTK92/next-boards/main/screenshot.png> =300x)

### 苦労した点

ログインしたユーザーのみがスレッドを観覧できるようにするには、どうすればいいのかという点で苦労しました。
https://next-auth.js.org/tutorials/securing-pages-and-api-routes
`unstable_getServerSession`を使用することで、APIを使用する際にログインしているかどうかを判定できるようになりました。
