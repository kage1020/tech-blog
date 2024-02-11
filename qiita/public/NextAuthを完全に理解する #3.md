---
title: 'NextAuthを完全に理解する #3'
tags:
  - JavaScript
  - 初心者
  - Webアプリケーション
  - Next.js
  - next-auth
private: false
updated_at: '2023-08-03T15:28:54+09:00'
id: 195fdd8749f2439849c1
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

最近，Next.jsを使ったwebアプリケーションを作り始めました．この中でログイン機能を付けたくてNextAuth.jsの導入をしたのですが，無知すぎて何をしているのかわかりませんでした．そこで，勉強を兼ねてここにドキュメントの内容を日本語訳して記しておきます．

:::note warn
**2023/06/18 追記**
NextAuth.jsはAuth.jsに変わりました．Next.js App Routerの場合の実装も併せて新しく記事を作成したのでそちらをご覧ください．

https://qiita.com/kage1020/items/fca49e9b42b972b70b8c
:::

# この記事の内容

この記事はNextAuthのドキュメントを私なりに解釈して書き留めたものです．いくつか誤りがあると思いますのでコメント等で指摘していただくと幸いです．
ドキュメントを読みたい方は以下からどうぞ．

https://next-auth.js.org

前回の記事は以下からどうぞ．

https://qiita.com/kage1020/items/e5b0053d7046a9b1f628

今回は`[...nextauth].js`についてです．

:::note warn
~~2022/02/21現在の最新バージョンは4.2.1であり，この記事はその時のドキュメントを元に書いています．~~
2022/09/25 追記
最新バージョン4.10.3に更新しました．
:::

---
### 2022/03/27 [providers](#providers), [callbacks](#callbacks)について追記

# 初期化

`[...nextauth].js`ファイルでは，`/api/auth/*`で始まるすべてのAPIリクエストを処理します．いわゆるダイナミックルーティングです．
ほとんどの場合以下のような初期化で済みます．

```js:/pages/api/auth/[...nextauth].js
import NextAuth from "next-auth"

export default NextAuth({
  ...
})
```

初期化の途中でログをとりたい場合など特殊な条件下において以下のように初期化もできます．

```js:/pages/api/auth/[...nextauth].ts
import type { NextApiRequest, NextApiResponse } from "next"
import NextAuth from "next-auth"

export default async function auth(req: NextApiRequest, res: NextApiResponse) {
  // Do whatever you want here, before the request is passed down to `NextAuth`
  return await NextAuth(req, res, {
    ...
  })
}
```

ただし，`NextAuth`関数はレスポンスを閉じるのでその後のコードを実行することはできません．

:::note alert
NextAuthが機能するために必要なリクエストの一部を変更することは、デフォルトのクッキーをいじるように、予期せぬ結果をもたらす可能性があり、間違った操作をするとセキュリティホールをもたらす可能性があります。このような結果を理解した上で、変更するようにしてください。
:::

# Options

### providers

OAuthを用いてSNS等での認証をすることかできます．詳しい手順は以下のドキュメントに従ってください．

https://next-auth.js.org/configuration/providers/oauth#how-to

```js:/pages/api/auth/[...nextauth].js
import TwitterProvider from "next-auth/providers/"
...
providers: [
  TwitterProvider({
    clientId: process.env.TWITTER_ID,
    clientSecret: process.env.TWITTER_SECRET
  })
],
...
```

環境変数にIDとSECRETを指定するのも忘れずに

```:.env.local
TWITTER_ID=YOUR_TWITTER_CLIENT_ID
TWITTER_SECRET=YOUR_TWITTER_CLIENT_SECRET
```

また，メールでの認証も可能です．ただし，データベースは必須です．

```js:/pages/api/auth/[...nextauth].js
import EmailProvider from `next-auth/providers/email`
...
providers: [
  EmailProvider({
    // id: "myEmail", // Unique ID for the provider
    // name: myMail, // Descriptive name for the provider
    type: "email",
    server: process.env.EMAIL_SERVER,
    from: process.env.EMAIL_FROM,
    // sendVerificationRequest: Callback to execute when a verification request is sent
    // maxAge: 24 * 60 * 60, // How long email links are valid for (default 24h)
  }),
],
...
```

2要素認証などをする場合にはCredentialでの認証が便利です．

```js:pages/api/auth/[...nextauth].js
import CredentialsProvider from "next-auth/providers/credentials"
...
providers: [
  CredentialsProvider({
    // The name to display on the sign in form (e.g. 'Sign in with...')
    name: 'Credentials',
    id: "myCredential",
    type: "credentials",
    // The credentials is used to generate a suitable form on the sign in page.
    // You can specify whatever fields you are expecting to be submitted.
    // e.g. domain, username, password, 2FA token, etc.
    // You can pass any HTML attribute to the <input> tag through the object.
    credentials: {
      username: { label: "Username", type: "text", placeholder: "jsmith" },
      password: {  label: "Password", type: "password" }
    },
    async authorize(credentials, req) {
      // You need to provide your own logic here that takes the credentials
      // submitted and returns either a object representing a user or value
      // that is false/null if the credentials are invalid.
      // e.g. return { id: 1, name: 'J Smith', email: 'jsmith@example.com' }
      // You can also use the `req` object to obtain additional parameters
      // (i.e., the request IP address)
      const res = await fetch("/your/endpoint", {
        method: 'POST',
        body: JSON.stringify(credentials),
        headers: { "Content-Type": "application/json" }
      })
      const user = await res.json()

      // If no error and we have user data, return it
      if (res.ok && user) {
        return user
      }
      // Return null if user data could not be retrieved
      return null
    }
  })
]
...
```
`authorize`関数の`credentials`引数は直前に記述した`credentials`を参照できます．
また，`authorize`関数は[`User`](https://next-auth.js.org/adapters/models/#user)を返す必要があります．デフォルトでは`id`, `name`, `email`, `image`しか認識しないので，認証時に追加情報をもたせたい場合はカスタムプロバイダーで追加することに注意してください．

https://next-auth.js.org/adapters/models/#user

### secret

トークンのハッシュ化，Cookieの署名/暗号化，暗号鍵の生成に使われます．`NEXTAUTH_SECRET`を指定している場合，必須ではありません．linuxでは以下のコマンドで生成できます．

```
openssl rand -base64 32
```

:::note warn
この値を指定しない場合，警告が発生します．
:::

### pages

通常，サインインや認証エラーページはNextAuthによって自動生成されます．カスタムページで認証を行いたい場合は`pages`を設定します．

```js:/pages/api/auth/[...nextauth].js
...
  pages: {
    signIn: '/auth/signin',
    signOut: '/auth/signout',
    error: '/auth/error', // Error code passed in query string as ?error=
    verifyRequest: '/auth/verify-request', // (used for check email message)
    newUser: '/auth/new-user' // New users will be directed here on first sign in (leave the property out if not of interest)
  }
...
```

### theming

デフォルトテーマはシステム依存ですが，強制的にライトテーマやブランドカラーに変更することができます．

```js:/pages/api/auth/[...nextauth].js
theme: {
  colorScheme: "auto", // "auto" | "dark" | "light"
  brandColor: "", // Hex color code
  logo: "" // Absolute URL to image
}
```

### callbacks

```js:/pages/api/auth/[...nextauth].js
...
  callbacks: {
    async signIn({ user, account, profile, email, credentials }) {
      ...
    },
    async redirect({ url, baseUrl }) {
      ...
    },
    async session({ session, user, token }) {
      // jwtが呼ばれた後に実行されます．
      return { ...session, ...token } // JWTではuserは渡されません
    },
    async jwt({ token, user, account, profile, isNewUser }) {
      // authorize関数でオリジナル変数をuserにセットした場合，ここでtokenにその値を写す必要があります．
      token.idToken = user.idToken
      return token // 初回の認証以降，tokenしか渡されません．
    }
...
}
```

`signIn`関数はユーザーのサインインが許可された場合のみ実行されます．
`redirect`関数はcallback URLへリダイレクトされるたびに呼ばれます．
JWTを使用している場合，`useSession`や`getSession`が呼ばれるとき`jwt`関数が実行されます．

TypeScriptの場合，tokenやsessionに新しい値を持たせるときは型定義を拡張する必要があります．

```ts:/types/next-auth.d.ts
import NextAuth, { DefaultSession } from "next-auth"
import { JWT } from "next-auth/jwt"

declare module "next-auth" {
  /**
   * Returned by `useSession`, `getSession` and received as a prop on the `SessionProvider` React Context
   */
  interface Session {
    user: {
      /** The user's postal address. */
      address: string
    } & DefaultSession["user"]
  }
}

declare module "next-auth/jwt" {
  /** Returned by the `jwt` callback and `getToken`, when using JWT sessions */
  interface JWT {
    /** OpenID ID Token */
    idToken?: string
  }
}
```
```ts
...
  callbacks: {
    async session({ session, token }: { session: Session, token: JWT }) {
      // jwtが呼ばれた後に実行されます．
      session.user.address = token.address
      return session // JWTではuserは渡されません
    },
    async jwt({ token, user }: { token: JWT, user: User }) {
      // authorize関数でオリジナル変数をuserにセットした場合，ここでtokenにその値を写す必要があります．
      if (user) token.address = user.address
      return token // 初回の認証以降，tokenしか渡されません．
    }
...
}
```

### logger

外部のロギングサービスへはこのオプションから送ることができます．

```js
{
  ...
  logger: {
    error(code, metadata) {
      log.error(code, metadata)
    },
    warn(code) {
      log.warn(code)
    },
    debug(code, metadata) {
      log.debug(code, metadata)
    }
  }
}
```

### others

その他細々したオプションがありますがここでは割愛します．詳しくは以下のドキュメントを参照してください．

https://next-auth.js.org/configuration/options

# 環境変数

### NEXTAUTH_URL, NEXTAUTH_URL_INTERNAL, VERCEL_URL

開発環境，本番環境共に`NEXTAUTH_URL`にroot URLを指定してください．ただし，Vercelへデプロイする場合は必要ありません．また，サーバーがroot URLにアクセスできない環境の場合サーバーサイドの呼び出しは`NEXTAUTH_URL`の代わりに`NEXTAUTH_URL_INTERNAL`を使用します．デフォルトは`NEXTAUTH_URL`です．

2022/06/06 追記
Vercelにdeployする場合`VERCEL_URL`を指定する必要があるとの記事がいくつか散見されますが，ドキュメントには**指定しなければいけない**という文字は見当たらないので必須ではないと思います．(実際，筆者は設定せずに使えていますし，`VERCEL_URL`は予約語なのでわざわざ指定する必要はないはず．．．)

```:.env
NEXTAUTH_URL=https://example.com
NEXTAUTH_URL_INTERNAL=http://10.240.8.16
```

### NEXTAUTH_SECRET

JWTの暗号化，メール認証トークンのハッシュ化に使用します．これは[secret](#secret)のデフォルト値です．[secret](#secret)オプションは将来廃止され，この値が使われるかもしれません．

***

以上でドキュメントの重要な部分は説明できました．あとはデータベースとの連携であったり，warning，error，高度な設定などですので各自でドキュメントを読んでいただけると幸いです．もう一度ドキュメントを貼っておきます．

https://next-auth.js.org/

また，チュートリアルがまとめられているので一度目を通してみるといいかもしれないです．

https://next-auth.js.org/tutorials
