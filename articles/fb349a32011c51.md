---
title: "最近流行りのNext.js周りのアプリ構成テンプレート"
emoji: "🐷"
type: "tech"
topics:
  - "next"
  - "typescript"
  - "prisma"
  - "tailwind"
  - "nextauth"
published: true
published_at: "2023-04-05 17:31"
---

最近（？）流行りの技術を使ったNext.jsアプリのテンプレートを作成しました．ご自由にお使いください．
いわゆる「T3 Stack」をメインとした構成です．

https://github.com/kage1020/NextStackApp

https://zenn.dev/mikinovation/articles/20220911-t3-stack

# Next.js

`app`ディレクトリが導入されたりといまだに人気も開発速度も衰えないすごいヤツです．

https://nextjs.org/

# Tailwind CSS

ユーティリティファーストでスタイルを書くCSSフレームワーク．CSSをそれなりに理解していないと扱いずらかったり，JSXに直接だらだらと書き込むので毛嫌いする人がいたりしますが，慣れれば１ファイルで完結するのでとても楽です．
こちらも，モダンCSSへの対応がそれなりに早いのでstyled-componentsやcss modulesなどと肩を並べていると思っています．

https://tailwindcss.com/

# TypeScript

当 た り 前

https://www.typescriptlang.org/

# NextAuth.js(Auth.js)

Next.jsだけでフロントもバックも作るんだったらこれ以外の選択肢はないくらいには最強のライブラリ．これの強みは，たくさんのSNS認証が４行書くだけでできること．これ以上に勝るものはない．

https://next-auth.js.org/

# Prisma

バックエンドとデータベースをつなぐためのORM．Next.jsでバックエンドを作るとき，フロントからデータベースまで同じ型を使うことができるため利便性がヤバい（語彙力）．また，クエリをTypeScriptで書くため，CRUD操作に迷うことがないのは楽．

https://www.prisma.io/

本家の「T3 Stack」では[tRPC](https://trpc.io/)を使っているが，正直Prismaの同じ型を使っていれば不要だと思う．Next.jsにServer Actionsが追加されたこともあり，内部APIだけならもっと柔軟なサーバーコードの設計ができるようになると思います．なので，今回は導入を見送り．

```ts:Prismaの型をそのまま使う
// components/LoginForm.tsx
import { User as PrismaUser } from '@prisma/client'

type User = Pick<PrismaUser, 'id' | 'email' | 'name'>
export type LoginSchema = z.infer<typeof LoginSchema>

export default function LoginForm() {
  onSubmit = async () => {
    const res = await axios.post<LoginSchema, User>('/api/user', {
      email,
      password
    })
    // res は User 型
    ...
  }
}
```

```ts:Server Actions
// actions/user.ts
"use server"

export const getUserData = async (email: string, password: string) => {
  return await prisma.user.findUnique({
    where: {
      email,
      password
    },
    select: {
      id: true,
      email: true,
      name: true
    }
  })
}

// components/LoginForm.tsx
"use client"

export default function LoginForm() {
  onSubmit = async () => {
    const res = await getUserData(email, password)
    // res は User 型
  }
}
```

外部向けのAPIは引き続きAPI Routesを使うといいと思います．そのときにはtRPCは役に立つかもしれません．

# SWR

これは誰が何と言おうと「データフェッチライブラリ兼状態管理ライブラリ」です（異論は認める）．reduxやrecoilは規模が大きくなったら検討すればいいので，最初期であんな面倒なものを組む必要はないと思います．SWRはkey-valueでデータをキャッシュするので，フェッチしたデータをクエリURLごとにキャッシュしたり，stateを文字列と一緒にグローバルで保管したりできます．

https://swr.vercel.app/ja

https://zenn.dev/itizawa/articles/9f71e1f636d3d2

# next-pwa

iOSが通知機能をサポートしたことで，今後需要が大きく拡大しそうなので入れました．`next.config.js`に２行追加するだけなのでとても楽．
ただ，最近更新が止まっているので新しいのを探したほうがいいかもしれない．

https://github.com/shadowwalker/next-pwa

いつかNext.js公式がサポートするかも？

https://github.com/vercel/next.js/discussions/42676

現状アクティブなフォークは以下のレポジトリです．

https://github.com/DuCanhGH/next-pwa

上に挙げたライブラリたちが以下のライブラリに集約されていきそうです．

https://github.com/serwist/serwist

# あとがき

その他たくさんのライブラリが入っていますが，一度は見たことあるであろう者たちばかりですので説明は割愛します．

この構成が皆さんのアプリ構成の一助となれば幸いです．
