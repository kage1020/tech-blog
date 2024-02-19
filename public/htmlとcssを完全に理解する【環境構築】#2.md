---
title: htmlとcssを完全に理解する【環境構築】#2
tags:
  - HTML
  - CSS
  - 初心者
private: false
updated_at: '2021-11-12T11:26:15+09:00'
id: 66bbdac4b26068b78146
organization_url_name: null
slide: false
ignorePublish: false
---
#目的
このシリーズではhtmlとcssを完全に理解するまでの道筋を大まかに書いていきます．対象は初めてウェブページを作成したいけど何をすればよいかわからない人です．このシリーズを読み終わるころには，あなたはウェブページ作成マスターだ！
各記事は以下の表から飛べます．

|       | html編                      |       | css編                              |
|:-----:|:---------------------------:|:-----:|:----------------------------------:|
| 第1回 | [導入](https://qiita.com/kage1020/items/7b646e85b73d68569862)| 第9回 | 文字 (coming soon)                 |
| 第2回 | 環境構築                     | 第10回 | 罫線 (coming soon)                |
| 第3回 | [タグ Part1](https://qiita.com/kage1020/items/b5c833cb928934d0d1e6)| 第11回 | inlineとblock (coming soon)      |
| 第4回 | タグ Part2(coming soon)      | 第12回 | リスト (coming soon)              |
| 第5回 | 画像 (coming soon)           | 第13回 | テーブル(表) (coming soon)        |
| 第6回 | テーブル(表) (coming soon)   | 第14回 | レスポンシブデザイン (coming soon) |
| 第7回 | リンク (coming soon)         | 第15回 | 各ブラウザへの対応 (coming soon)   |
| 第8回 | レイアウト (coming soon)     |

#環境構築

プログラミングをする上で**環境構築は重要**です．よりよい環境を構築することでプログラムを書く時間は大幅に減少し，ストレスフリーに楽しく書くことができます．自分に合った環境を手に入れるのに1日を費やしてもかまいません．

###Windows User

windowsを使っている人はMicrosoft loverなので(偏見)，**VisualStudioCode**(以下，VSCode)を使うことを強く推奨します．VSCodeはMicrosoftが提供しているテキストエディタであり，自由なカスタマイズを特徴とした無料で使えるツールです．VSCodeのインストールは以下のリンクから．

[Visual Studio Code – コード エディター | Microsoft Azure](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)

詳しくは[この記事](https://qiita.com/psychoroid/items/7d85ae6bade4a67aedb1) [^1] などを参考にするといいと思います．

[^1]: [Visual Studio Code (Windows版) のインストール](https://qiita.com/psychoroid/items/7d85ae6bade4a67aedb1)

<details>
<summary>私が実際に使用している拡張機能はコチラ</summary><div>

####Japanese Language Pack for Visual Studio Code(推奨)

VSCodeを日本語化します．
https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja

####Material Icon Theme

拡張子ごとにファイル名の横にアイコンを表示してくれます．
https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme

####indent-rainbow

インデントの深さごとに色分けしてくれます．マジ便利
https://marketplace.visualstudio.com/items?itemName=oderwat.indent-rainbow
インデントについては[この記事](https://qiita.com/kawaemon/items/25ddaedd3afb61d7d014) [^2] を読むといいと思います．

[^2]: [インデントは大事だよっていう話](https://qiita.com/kawaemon/items/25ddaedd3afb61d7d014)

####Code Spell Checker

タイピングミスを指摘してくれます．
https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker

####Visual Studio IntelliCode

「もしかしてこれを書きたいんじゃない？」って候補を表示してくれる賢い子です．
https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode

####Live Server(推奨)

htmlやcssファイルに変更を加えたとき，リロードせずとも表示してくれます．
https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer

####Bracket Pair Colorizer 2

今回はあまり出番はないですが，「この}はどの{と対応しているんだ！」なイライラを失くせます．
https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer-2

####HTML Snippets

htmlを書きやすくしてくれます．
https://marketplace.visualstudio.com/items?itemName=abusaidm.html-snippets

####HTML CSS Support

cssを見やすく色分けしてくれます．
https://marketplace.visualstudio.com/items?itemName=ecmel.vscode-html-css

####IntelliSense for CSS class names in HTML

classなどの名前をいい感じに提案してくれます．
https://marketplace.visualstudio.com/items?itemName=Zignd.html-css-class-completion

</div></details>

###Mac User

私は宗教上の理由によりApple製品を携帯していませんので，どのエディタがいいなどはわかりません．迷ったらVSCodeを使えばいいと思います(布教)．以下にいくつかエディタの候補を挙げておきます．

* CotEditor
* DreamWeaver
* Sublime Text
* Atom

#実行環境

このシリーズでは以下の実行環境で動作を確認しています．それ以外の環境では正常に動作しない場合がありますので，コメントにてお知らせいただくと幸いです．

* windows 10
  * ブラウザ:Google Chrome (version 95.0.4638.69)
  * Microsoft Edge (version 95.0.1020.44)

##参考文献
高橋朋代，森智佳子，"作りながら学ぶ HTML/CSSデザインの教科書"，SBクリエイティブ，2019年

<br>

次回はhtml編1回目，タグについて理解していきます．たぐとはなんぞや？の頭で読んでください．
[htmlとcssを完全に理解する【導入】#1](https://qiita.com/kage1020/items/7b646e85b73d68569862)←前回 &emsp;&emsp;次回→[htmlとcssを完全に理解する【タグ Part1】#3](https://qiita.com/kage1020/items/b5c833cb928934d0d1e6)
