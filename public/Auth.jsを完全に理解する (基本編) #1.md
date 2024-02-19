---
title: 'Auth.jsを完全に理解する (基本編) #1'
tags:
  - authentication
  - 認証
  - Next.js
  - nextauth.js
  - Auth.js
private: false
updated_at: '2023-12-11T13:07:03+09:00'
id: fca49e9b42b972b70b8c
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

この記事はAuth.jsがどのようなものか，どのように実装すればいいかなどをドキュメントを要約しながら紹介するものです．

https://authjs.dev/

:::note warn
Auth.jsは2023/06/11現在ドキュメント整備中です．現在のドキュメントとは内容が異なる場合があります．この記事では[旧ドキュメント](https://next-auth.js.org/)の内容も交えて解説しています．
:::

# Auth.jsとは？

![auth-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2274081/c935bfc6-d4ea-6a6e-e8ae-fbf844fe688c.png)
(<https://authjs.dev/> より)

> Auth.js is a complete open-source authentication solution for web applications. Check out the live demos of Auth.js in action:
> * [Next.js](https://next-auth-example.vercel.app/)
> * [SvelteKit](https://sveltekit-auth-example.vercel.app/)
> * [SolidStart](https://auth-solid.vercel.app/)

Auth.jsはwebアプリのための完全なオープンソース認証ソリューションです．その特徴として，
* 簡単に OAuth と OpenID Connect を実装できる
* モダンなフレームワークやデータベースでランタイム環境に関係なく柔軟な対応
* 安全な設計によるセキュリティ

があげられます．

OAuth認証や二要素認証が必須となってきている現在では，簡単で柔軟なケースに対応できるとても便利なツールだと思います．

## 認証の種類

Auth.jsでは以下の3つの認証方式が実装できます．

* OAuth authentication
* Email authentication
* Credentials authentication

順に説明していきます．

### OAuth & OpenID Connect authentication

一番簡単なのはOAuth認証です．GoogleやTwitterを用いて代わりに認可・認証してもらうことで，アプリ自体はパスワード等を持たずにユーザー識別ができます．また，JWT(後述)を使うことでデータベースを持つことなく安全な運用ができます．

OAuthやOpenID Connect，認可・認証の違い等については以下の記事などがわかりやすいと思います．

https://qiita.com/TakahikoKawasaki/items/e37caf50776e00e733be

https://qiita.com/TakahikoKawasaki/items/498ca08bbfcc341691fe

対応しているプロバイダは50個以上ありそれなりの知名度のサービスなら使えると思います．

:arrow_down: 対応プロバイダ一覧

https://next-auth.js.org/configuration/providers/oauth#built-in-providers

### Email authentication

文字通り登録メールアドレスを用いて認可を行います．メールアドレスデータベースをもとにマジックリンクをリクエストしたアドレスに送信し，パスワードレスでセッションを渡します．データベースが必要だったり，セキュリティレベルの高いメールサーバを使う必要があったりと欠点がいくつかありますが，WebAuthnと組み合わせるなどすれば十分に活用できると思います．

### Credentials authentication

よくあるようなIDとパスワードで認可するタイプの方法です．パスワードをデータベースに抱えるリスクと総当たり攻撃による危険性があるため推奨はされていないです．また，Auth.jsもあえて拡張性を下げた実装をしているようです．

## データベースの種類

Auth.jsは認証周りの機能を提供するライブラリであり，データベースやORMとバックエンドの連携機能も提供されています．連携できるデータベースは以下のとおりです．

* MySQL
* MariaDB
* Postgres
* MongoDB
* SQLite

また，これらのデータベースをAuth.js側で処理するためのアダプタが用意されています．

* Dgraph
* DynamoDB
* Fauna
* Firebase
* Mikro ORM
* MongoDB
* Neo4j
* PouchDB
* Prisma
* Sequelize
* Supabase
* TypeORM
* Upstash
* Xata

ここに対応するデータベースやORMがなくとも，自作のアダプタを作成することで使えるようになります．

https://authjs.dev/guides/adapters/creating-a-database-adapter

## JWTとは？

Json Web Tokenのことで，Base64エンコードされたヘッダ・ペイロードを公開鍵暗号方式で署名することで改竄なく認証情報をやり取りすることができる仕組みです．

例えば，ヘッダにアルゴリズムとトークンタイプを設定し，

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

ペイロードにユーザーID，ユーザー名，作成日時を設定します．

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

これらをBase64エンコードしたうえで暗号化します．`secret`は任意の文字列ですが，256ビットなど十分な長さが推奨です．

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

そして，Base64エンコードされた値と暗号化した値を`.`でつなぎ合わせたのがJWTです．

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
.XbPfbIHMI6arZ3Y922BhjWgQzWXcXNrz0ogtVhfEd2o
```

## おわりに

以上がAuth.jsの基本的な内容となります．次回以降はAuth.jsを実際に実装していきたいと思います．

https://qiita.com/kage1020/items/8224efd0f3557256c541

## 参考文献

https://jwt.io/

https://qiita.com/knaot0/items/8427918564400968bd2b
