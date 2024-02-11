---
title: 'NextAuthを完全に理解する #2'
tags:
  - JavaScript
  - 初心者
  - Webアプリケーション
  - Next.js
  - next-auth
private: false
updated_at: '2023-08-03T15:27:52+09:00'
id: e5b0053d7046a9b1f628
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

前回の記事は以下からどうぞ．

https://qiita.com/kage1020/items/bdefabcd09d86e78d474

今回はclient sideについてです．

:::note warn
~~2022/02/21現在の最新バージョンは4.2.1であり，この記事はその時のドキュメントを元に書いています．~~
2022/09/25 追記
最新バージョン4.10.3に更新しました．
:::

# Sessionの取得

ページやコンポーネント内でセッションを取得するには`useSession` Hookを使います．

```jsx:example
import { useSession } from "next-auth/react"

export default function Component() {
  const { data: session, status } = useSession()

  if (status === "authenticated") {
    return <p>Signed in as {session.user.email}</p>
  }

  return <a href="/api/auth/signin">Sign in</a>
}
```

`useSession`は`data`と`status`を返します．`data`の型は

* fetch前は`undefined`
* fetchに失敗した場合は`null`
* fetchに成功した場合は`Session`

です．`Session`オブジェクトは

```typescript:Session
{
  user?: {
    name?: string | null
    email?: string | null
    image?: string | null
  }
  expires?: string
}
```

`status`の型は`"loading" | "authenticated" | "unauthenticated"`となります．次回の記事にてこの`Session`の拡張方法を記載します．

# サインインしていない場合の挙動

上手く理解できなかったので原文を貼ります．

>Due to the way how Next.js handles getServerSideProps and getInitialProps, every protected page load has to make a server-side request to check if the session is valid and then generate the requested page (SSR). This increases server load, and if you are good with making the requests from the client, there is an alternative. You can use useSession in a way that makes sure you always have a valid session. If after the initial loading state there was no session found, you can define the appropriate action to respond.
>
>The default behavior is to redirect the user to the sign-in page, from where - after a successful login - they will be sent back to the page they started on. You can also define an onFail() callback, if you would like to do something else:

```javascript
import { useSession } from "next-auth/react"

export default function Admin() {
  const { status } = useSession({
    required: true,
    onUnauthenticated() {
      // The user is not authenticated, handle it here.
    }
  })

  if (status === "loading") {
    return "Loading or not authenticated..."
  }

  return "User is logged in"
}
```

`onUnauthenticated()`でsign-inページへのリダイレクトを防げる？

# Custom Client Session Handling

`getServerSideProps`はクエリごとにセッションが有効かを判定するため通信回数が増える．そこでコンポーネントに`key-value`を与えることで判定回数を少なくできる．

```jsx:anyComponent.js
export default function AnyComponent() {
  const { data: session } = useSession()
  return <childComponent />
}

AnyComponent.auth = {
  role: "admin",
  loading: <AnyComponentLoadingSkeleton />,
  unauthorized: "/login-with-different-user", // redirect to this url
}
```
```jsx:_app.js
export default function App({
  Component,
  pageProps: { session, ...pageProps },
}) {
  return (
    <SessionProvider session={session}>
      {Component.auth.role === "admin" ? (
        <Admin>
          <Component {...pageProps} />
        </Admin>
      ) : (
        <Client>
          <Component {...pageProps} />
        </Client>
      )}
    </SessionProvider>
  )
}
```

# getSession

`useSession`のサーバー版です．正確には，React(Next.js)の外側でセッションが欲しいときに使います．

```jsx:example
async function myFunction() {
  const session = await getSession()
  /* ... */
}
```

:::note warn
v4.10.3現在，`getSession`は非推奨であり，安定版である`getServerSession`への移行が進められています．詳しくは [unstable_getServerSession](https://next-auth.js.org/configuration/nextjs#unstable_getserversession) を見てください．
:::

# getCsrfToken

オリジナルのサインインページを作成するときに使います．後述するビルトインの`singIn`や`signOut`関数では自動で`Cross Site Request Forgery Token(CSRF Token)`が付与されますが，自前で`/api/auth/signin`などにPOSTする場合はcsrfTokenが必要です．
クライアント，サーバーどちらも使えます．

```jsx:client
async function myFunction() {
  const csrfToken = await getCsrfToken()
  /* ... */
}
```
```jsx:server
import { getCsrfToken } from "next-auth/react"

export default async (req, res) => {
  const csrfToken = await getCsrfToken({ req })
  /* ... */
  res.end()
}
```

# getProviders

自前でサインインページを作っているとき，SNS等でログインさせたいときがあると思います．こんな時にこの関数で`[...nextauth].js`に設定したプロバイダーを取得できます．Objectで取得できます．
クライアント，サーバーどちらも使えます．
```jsx:client
async function myFunction() {
  const providers = await getProviders()
  /* ... */
}
```
```jsx:server
import { getProviders } from "next-auth/react"

export default async (req, res) => {
  const providers = await getProviders()
  console.log("Providers", providers)
  res.end()
}
```
```js
{
  github: {
    id: "github",
    name: "GitHub",
    type: "oauth",
    signinUrl: "http://localhost:3000/api/auth/signin/github",
    callbackUrl: "http://localhost:3000/api/auth/callback/github"
  },
  ...
}
```

# signIn・signOut

NextAuth.jsが標準で提供するサインイン・サインアウト関数です．

```jsx:example
import { signIn, signOut } from "next-auth/react"

export default () => <button onClick={() => signIn()}>Sign in</button>
// or 
export default () => (
  <button onClick={() => signIn("google")}>Sign in with Google</button>
)
// or 
export default ({ email }) => (
  <button onClick={() => signIn("email", { email })}>Sign in with Email</button>
)

export default () => <button onClick={() => signOut()}>Sign out</button>
```

また，第2引数に認証後のリダイレクト先を指定することもできます．

```js:example
signIn(undefined, { callbackUrl: 'http://localhost:3000/foo' })
signOut({ callbackUrl: 'http://localhost:3000/foo' })
```

`email`と`credentials`での認証のみにおいてリダイレクトをせずにPromiseを取得できます．

```js:example
const res = signIn('credentials', { redirect: false, password: 'password' })

// resの中身
{
  error: string | undefined
  status: number
  ok: boolean
  url: string | null
}
```

# sessionProvider

前回も説明した通り，ページやコンポーネントの中でセッションを使いたい場合は`sessionProvider`でラップします．ここで取得したセッションはタブごとに保存され，全てのタブで同期されます．また，`localStorage`や`sessionStorage`に保存されることはありません．(Cookieに保存されます．)

```jsx:example
import { SessionProvider } from "next-auth/react"

export default function App({
  Component,
  pageProps: { session, ...pageProps },
}) {
  return (
    <SessionProvider
      session={session}
      // Re-fetch session every 5 minutes
      refetchInterval={5 * 60}
      // Re-fetches session when window is focused
      refetchOnWindowFocus={true}
    >
      <Component {...pageProps} />
    </SessionProvider>
  )
}
```

このとき，各ページでは以下の挙動と同義です．

```js:example
export async function getServerSideProps(ctx) {
  return {
    props: {
      session: await getSession(ctx)
    }
  }
}
```

:::note warn
Next.js@12.3.0以降でTypeScriptを使う場合，`_app.tsx`は以下のように書き換える必要があります．
```tsx
import type { AppProps } from 'next/app'
import { Session } from 'next-auth'
import { SessionProvider } from "next-auth/react"

export default function App({
  Component,
  pageProps: { session, ...pageProps },
}: AppProps<{ session: Session }>) {
  return (
    <SessionProvider
      session={session}
      // Re-fetch session every 5 minutes
      refetchInterval={5 * 60}
      // Re-fetches session when window is focused
      refetchOnWindowFocus={true}
    >
      <Component {...pageProps} />
    </SessionProvider>
  )
}
```

https://github.com/vercel/next.js/issues/40371#issuecomment-1241817292

:::

---

今回はここまで．次回は`[...nextauth].js`について書いていきます．

https://qiita.com/kage1020/items/195fdd8749f2439849c1
