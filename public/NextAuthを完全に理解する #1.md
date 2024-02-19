---
title: 'NextAuthを完全に理解する #1'
tags:
  - JavaScript
  - 初心者
  - Webアプリケーション
  - Next.js
  - next-auth
private: false
updated_at: '2023-08-03T15:28:23+09:00'
id: bdefabcd09d86e78d474
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

最近，Next.jsを使ったwebアプリケーションを作り始めました．この中でログイン機能を付けたくてNextAuth.jsの導入をしたのですが，無知すぎて何をしているのかわかりませんでした．そこで，勉強を兼ねてここにドキュメントの内容を日本語訳して記しておきます．

:::note warn
**2023/06/11 追記**
NextAuth.jsはAuth.jsに変わりました．Next.js App Routerの場合の実装も併せて新しく記事を作成したのでそちらをご覧ください．

https://qiita.com/kage1020/items/fca49e9b42b972b70b8c
:::

# この記事の内容

この記事はNextAuthのドキュメントを私なりに解釈して書き留めたものです．いくつか誤りがあると思いますのでコメント等で指摘していただくと幸いです．
ドキュメントを読みたい方は以下からどうぞ．

https://next-auth.js.org

:::note warn
~~2022/02/21現在の最新バージョンは4.2.1であり，この記事はその時のドキュメントを元に書いています．~~
2022/09/25 追記
最新バージョン4.10.3に更新しました．
:::

# NextAuthとは

>NextAuth.js is a complete open-source authentication solution for Next.js applications.
>It is designed from the ground up to support Next.js and Serverless.

Next.jsやSeverlessアプリのための完全なオープンソース認証ライブラリとのこと．
また，以下の3点が特徴のようです．

* フレキシブルで使いやすい
* データベースを必要としないデータの保持
* 初期設定がセキュア

1番目について，OAuthだけでなくSNSでの認証も可能なので柔軟な設計ができるのが魅力です．
2番目について，OAuth+JWTによって個人情報を保持する必要がないので楽に設計できます．もちろん，データベースでの管理も可能です．MySQL,SQLite等多くのデータベースに対応しています．

# NextAuthを始めよう

公式レポジトリにスターターキットがあるのでこちらも参照してください．

https://github.com/nextauthjs/next-auth-example

Next.jsのアプリケーションを既に作成している前提で始めます．インストールから

```bash
npm install next-auth
# or
yarn add next-auth
```

`/pages/api/auth`に`[...nextauth].js`を作成します．Next.jsを触っていればわかると思いますが，ダイナミックルーティングですべての設定がここで反映されます．中身を以下に示しますが，各自適宜変更してください．

```js:[...nextauth].js
import NextAuth from "next-auth"
import GithubProvider from "next-auth/providers/github"

export default NextAuth({
  // Configure one or more authentication providers
  providers: [
    GithubProvider({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    // ...add more providers here
  ],
})
```

プロジェクトのルートディレクトリに`.env.local`を作成し，以下のように書きます．あくまで一例ですので内容は適宜変更してください．

```
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=5ZzepQsBemOhhgDBcAA+bhcDFsMTz7AhznIfdZVvMHY=

GITHUB_ID=your_github_id_for_auth
GITHUB_SECRET=your_github_secret_for_auth
```

GithubのIDとsecretは以下でappを作成すると取得できます．clientとprivateを間違えないようにしてください．

https://github.com/settings/apps

認証機能を使えるようにするために，`SessionProvider`を設定します．

```jsx:/pages/_app.js
import { SessionProvider } from "next-auth/react"

export default function App({
  Component,
  pageProps: { session, ...pageProps },
}) {
  return (
    <SessionProvider session={session}>
      <Component {...pageProps} />
    </SessionProvider>
  )
}
```

`<Component/>`以下で`session`が使えるようになります．

```jsx
import { useSession } from "next-auth/react"

export default function AnyComponent() {
  const { data: session } = useSession()
  if (session) return <div>Signed in as {session.user.name}</div>
  else return <div>Not signed in</div>
}
```

***

今回は導入まで．次回以降で内部をじっくり記述していきます．

https://qiita.com/kage1020/items/e5b0053d7046a9b1f628
