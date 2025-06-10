---
title: "Next.js+TypeScriptにおけるmonorepoのtips"
emoji: "😺"
type: "tech"
topics:
  - "nextjs"
  - "typescript"
  - "vercel"
  - "monorepo"
published: true
published_at: "2022-10-16 15:28"
---

最近monorepoなるものを知ったので，Next.js+TypeScriptで構築する際の詰まった所などを書き残す．

# monorepoとは

monorepoについて簡単に自分の理解を示す．

* 複数のプロジェクトを同一のレポジトリで管理できて便利
* テストやlinterなどを統合できるからメンテも楽
* 開発者全員が全体把握しやすい

などなど．

# Projectの構築

基本的には以下のテンプレートをcloneすればいい．

https://github.com/vercel/next.js/tree/canary/examples/with-zones

---
2022/10/17 追記
TypeScript用テンプレートを作りました．

https://github.com/kage1020/with-zones-ts-app

---

基本的なディレクトリ構成図は以下．

```
root+
    |-blog+ // sub app
    |     |-public
    |     |-src
    |     |-.eslintrc.json
    |     |-next.config.js
    |     |-package.json
    |     |tsconfig.json
    |     ...
    |-home+ // main app
    |     |-public
    |     |-src
    |     |-.eslintrc.json
    |     |-next.config.js
    |     |-package.json
    |     |tsconfig.json
    |     ...
    |-.gitignore
    |-.env
    |-.eslintrc.json
    |-README.md
    |-package.json
    ...
```

特に特殊なものはない．

問題なのは`.eslintrc.json`の設定．このままVercelでデプロイしようとすると，`no-html-for-link-pages`のエラーを吐く．

https://nextjs.org/docs/messages/no-html-link-for-pages

これは，sub appの中でsub appのルートとmain appのルートをnext/linkと`<a>`タグを使い分けているために，どっちかにしろよと怒られているためである．

```tsx:sub appのindex.tsx
<Link href="/"> {/* sub appのroot */}
  <a>Go to Top</a>
</Link>

<a href="/">Go to Home</a> {/* main appのroot */}
```

これを解決するには，sub appでのaタグによるルート指定をmain appから継承すればいい．
Next.jsは専用の`eslint-plugin-next`というrulesが存在するため，main appのrulesをsub appに継承し，sub appにおける`/`がmain appにおける`/`を指すようにする．

```json:/blog/.eslintrc.json
{
  "extends": ["../home/eslintrc.json", "next/core-web-vitals"],
}
```

Vercelでbuildログを見ると

```
error - ESLint: Failed to load config "../home/eslintrc.json" to extend from. Referenced from: /vercel/path0/chat/.eslintrc.json
```

のエラーが出てるが，問題なく動いているのでとりあえず大丈夫っぽい．

また，main appの`next.config.js`でroutingを制御していることがわかるが，これはNext.jsやmonorepoだけに制限しているわけではない．

```js
const { BLOG_URL } = process.env;

/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  async rewrites() {
    return [
      {
        source: '/:path*',
        destination: `/:path*`,
      },
      {
        source: '/blog',
        destination: `${BLOG_URL}/blog`,
      },
      {
        source: '/blog/:path*',
        destination: `${BLOG_URL}/blog/:path*`,
      },
    ];
  },
};

module.exports = nextConfig
```

`BLOG_URL`を全く関係ない別のアプリURLにしたり，NuxtやSvelteへのURLにしてもいい．
つまり，実質monorepoでなくても動く．これは以下のようなときに役立つかもしれない．

* 開発チームが全くの別物だけど，ドメインは同じものを使いたい．つまり，レポジトリは別々だけど同じサービスの一部として開発したい．
* monorepoだと規模が大きくてcloneがつらい．
* 他のアプリのissueやPRを表示したくない．(labelで管理すれば．．．)

などなど．

:::message
Vercelで無料利用枠でやる場合monorepoは1つのレポジトリに全部で3つまでなので注意．

また，configを見ればわかるが，main appで`/blog.tsx`を作ってもroutingはsub appの方に流れるので名前かぶりはできません．
:::

## おまけ

NextAuth.jsはmonorepoに対してもsessionが有効なので，各アプリ内で`SessionProvider`を設定していれば，main appで認証後，sub appでその認証情報を取得できます．
従って，`clientId`や`clientSecret`は1つ取得すれば十分であり，callbackも`http://localhost:3000`などroot URLに設定すれば大丈夫そうです．
