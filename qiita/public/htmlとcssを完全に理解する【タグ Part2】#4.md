---
title: htmlとcssを完全に理解する【タグ Part2】#4
tags:
  - ''
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---

# 目的

このシリーズではhtmlとcssを完全に理解するまでの道筋を大まかに書いていきます．対象は初めてウェブページを作成したいけど何をすればよいかわからない人です．このシリーズを読み終わるころには，あなたはウェブページ作成マスターだ！
各記事は以下の表から飛べます．

|       | html編                      |       | css編                              |
|:-----:|:---------------------------:|:-----:|:----------------------------------:|
| 第1回 | [導入](https://qiita.com/kage1020/items/7b646e85b73d68569862)| 第9回 | 文字 (coming soon)                 |
| 第2回 | [環境構築](https://qiita.com/kage1020/items/66bbdac4b26068b78146)| 第10回 | 罫線 (coming soon)                |
| 第3回 | [タグ Part1](https://qiita.com/kage1020/items/b5c833cb928934d0d1e6)| 第11回 | inlineとblock (coming soon)      |
| 第4回 | タグ Part2                   | 第12回 | リスト (coming soon)              |
| 第5回 | 画像 (coming soon)           | 第13回 | テーブル(表) (coming soon)        |
| 第6回 | テーブル(表) (coming soon)   | 第14回 | レスポンシブデザイン (coming soon) |
| 第7回 | リンク (coming soon)         | 第15回 | 各ブラウザへの対応 (coming soon)   |
| 第8回 | レイアウト (coming soon)     |

# 実行環境

このシリーズでは以下の実行環境で動作を確認しています．それ以外の環境では正常に動作しない場合がありますので，コメントにてお知らせいただくと幸いです．

* windows 10
  * Google Chrome (version 95.0.4638.69)
  * Microsoft Edge (version 95.0.1020.44)

また，解説はwindowsを使用していることを前提として記述しています．macの人はゴメンナサイ．

# 各タグの解説

以下に良く使う(かもしれない)タグの大まかな説明をしていきます．

### `<article>~</article>`

`<body>`の中で独立して完結した内容を表示する場合に使用する．ブログなどを書くとき各回の内容は`<article>`で囲む．[ドキュメント](https://developer.mozilla.org/ja/docs/Web/HTML/Element/article)を見るとコメントを囲ってもいいらしい．．．

### `<section>~</section>`

タイトルを含む小段落を表すのに使う．この解説もタグごとに`<section>`で囲むといいのかも．あまり使い道はない？

### `<nav>~</nav>`

日本語で言うところの目次または索引．よく画面の上側に常に表示されてたりする．
