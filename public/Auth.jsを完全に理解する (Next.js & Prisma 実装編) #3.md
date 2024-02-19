---
title: Auth.jsを完全に理解する (Next.js & Prisma 実装編) #3
tags:
  - Auth.js
  - nextauth.js
  - Next.js
  - TypeScript
  - 認証
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---

# はじめに

この記事はAuth.jsがどのようなものか，どのように実装すればいいかなどをドキュメントを要約しながら紹介するものです．

https://authjs.dev/

:::note warn
Auth.jsは2023/11/01現在ドキュメント整備中です．現在のドキュメントとは内容が異なる場合があります．この記事では旧ドキュメントの内容も交えて解説しています．
:::

今回はNext.js & Prisma 実装編です．前回の記事はこちら

https://qiita.com/kage1020/items/8224efd0f3557256c541

# Prismaとは

https://www.prisma.io/

> Prisma is an open source next-generation ORM. It consists of the following parts:

オープンソースの次世代ORM．ORMとはObject-Relational Mappingの略で，オブジェクト指向でRDBを操作できるようにするためのツールです．つまり，直接SQL文を書くのではなく，jsオブジェクトを介してクエリを投げられるようにするためのSQL wrapperです．

また，PrismaはTypeScriptをサポートしており，データベーススキーマから自動で型を生成し，型つきオブジェクトとしてTypeScriptのままデータベースを操作できます．

この記事では，Auth.jsのセッション情報管理をPrismaにスムーズ受け渡すための具体的な実装方法について紹介します．

# Prismaのインストール

[前の記事](https://qiita.com/kage1020/items/8224efd0f3557256c541)でNext.jsおよびAuth.jsのセットアップが完了していることを前提として始めます．

まず，Prismaをインストールしましょう．

```bash
npm i -D prisma
```

次にデータベースを設計します．今回はSQLiteで作成します．

```bash
npx prisma init --datasource-provider sqlite
```

すると，rootディレクトリに`prisma/schema.prisma`と`.env`が作成されます．その後，以下の変更を加えます．

- `prisma/schema.prisma`を`/src/prisma/schema.prisma`に移動し，`package.json`に追記
    ```json:package.json
    {
      "prisma": {
        "schema": "src/prisma/schema.prisma"
      },
    }
    ```
- `.gitignore`に`.env`を追加
