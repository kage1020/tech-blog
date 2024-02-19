---
title: htmlとcssを完全に理解する【導入】#1
tags:
  - HTML
  - CSS
  - 初心者
private: false
updated_at: '2024-01-06T20:22:48+09:00'
id: 7b646e85b73d68569862
organization_url_name: null
slide: false
ignorePublish: false
---
#目的
このシリーズではhtmlとcssを完全に理解するまでの道筋を大まかに書いていきます．対象は初めてウェブページを作成したいけど何をすればよいかわからない人です．このシリーズを読み終わるころには，あなたはウェブページ作成マスターだ！
各記事は以下の表から飛べます．

|       | html編                      |       | css編                              |
|:-----:|:---------------------------:|:-----:|:----------------------------------:|
| 第1回 | 導入                         | 第9回 | 文字 (coming soon)                 |
| 第2回 | [環境構築](https://qiita.com/kage1020/items/66bbdac4b26068b78146)| 第10回 | 罫線 (coming soon)                |
| 第3回 | [タグ Part1](https://qiita.com/kage1020/items/b5c833cb928934d0d1e6)| 第11回 | inlineとblock (coming soon)      |
| 第4回 | タグ Part2(coming soon)      | 第12回 | リスト (coming soon)              |
| 第5回 | 画像 (coming soon)           | 第13回 | テーブル(表) (coming soon)        |
| 第6回 | テーブル(表) (coming soon)   | 第14回 | レスポンシブデザイン (coming soon) |
| 第7回 | リンク (coming soon)         | 第15回 | 各ブラウザへの対応 (coming soon)   |
| 第8回 | レイアウト (coming soon)     |

#html is 何?

htmlとはHyperText Markup Language(ハイパーテキスト マークアップ ランゲージ)の略で．．．え？そんなのどうでもいい？
わかりました．簡単に言うと，「文字や画像を工夫して配置してなんかいい感じにしようぜ」言語です．
こんなやつ↓

```html
<html>
<body>
  <h1>タイトル<h1>
  <p>段落</p>
  <img scr="this-is-url">
</body>
</html>
```

#css is 何?

cssとはCascading Style Sheets(カスケーディング・スタイル・シート または カスケード・スタイル・シート)のりゃｋ(゜oﾟ(○=(-_-;
「文字や画像をなんかいい感じに装飾しようぜ」言語です．
こんなやつ↓

```css
body {
  margin: 0 auto;
  color: white;
}
```

***

全てのウェブページはこれら2つ(だけではない)を用いて作成されています．他にも様々な言語がウェブページ作成にかかわっていますが，本シリーズでは割愛します．

次回は実際にウェブページを作るためのの環境構築です．全15回の中で最も重要といっても過言ではないので，頑張りましょう．
次回→[htmlとcssを完全に理解する【環境構築】#2](https://qiita.com/kage1020/items/66bbdac4b26068b78146)
