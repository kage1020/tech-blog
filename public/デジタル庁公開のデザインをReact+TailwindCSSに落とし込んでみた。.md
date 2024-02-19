---
title: デジタル庁公開のデザインをReact+TailwindCSSに落とし込んでみた。
tags:
  - React
  - tailwindcss
  - デジタル庁
private: false
updated_at: '2023-03-03T00:56:02+09:00'
id: 83792343bace6781a35e
organization_url_name: null
slide: false
ignorePublish: false
---
以下の記事のパロディです．

https://qiita.com/aimufodna/items/099cba4ca5d19f813ab7

ソース元はこちら

https://www.digital.go.jp/policies/servicedesign/designsystem/

# 注意事項

* デザインをすべて正確に写したものではありません．
* なるべく実用に耐えるよう`props`等設定しましたが，あくまで参考程度にしてください．
* ラインセンスはFigmaにより CC BY 4.0 です．
* 何か間違い等あればご指摘いただければ幸いです．

# ソースコード・Demo

https://degital-agency-design-system-react.vercel.app/

https://github.com/kage1020/DegitalAgencyDesignSystem-React

コンポーネントをStorybookで公開しました．一部コンポーネントは正常に動作しないです．

https://main--6400c31e5deec0b7b6806583.chromatic.com

# コンポーネント詳細

## カラーパレット

<details>
<summary>tailwind.config.js</summary>

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'class',
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        forest: {
          DEFAULT: '#259D63',
          50: '#E6F5EC',
          100: '#C2E5D1',
          200: '#9BD4B5',
          300: '#71C598',
          400: '#51B883',
          500: '#2CAC6E',
          600: '#259D63',
          700: '#1D8B56',
          800: '#197A4B',
          900: '#115A36',
        },
        sea: {
          DEFAULT: '#003EE5',
          50: '#E8F1FE',
          100: '#C5D7FB',
          200: '#9DB7F9',
          300: '#7096F8',
          400: '#4979F5',
          500: '#0946F1',
          600: '#003EE5',
          700: '#0031D8',
          800: '#0024CE',
          900: '#0000BE',
          darken: {
            300: '#3F72F6',
            600: '#0030B2',
          },
        },
        sumi: {
          DEFAULT: '#1A1A1C',
          50: '#F8F8FB',
          100: '#F1F1F4',
          200: '#E8E8EB',
          300: '#D8D8DB',
          400: '#B4B4B7',
          500: '#949497',
          600: '#757578',
          700: '#626264',
          800: '#414143',
          900: '#1A1A1C',
        },
        sun: {
          DEFAULT: '#EC0000',
          50: '#FFE7E6',
          100: '#FFC8B8',
          200: '#FFA28B',
          300: '#FF7B5C',
          400: '#FF5838',
          500: '#FF4B36',
          600: '#FF220D',
          700: '#FA1606',
          800: '#EC0000',
          900: '#D50000',
        },
        wood: {
          DEFAULT: '#C16800',
          50: '#F8F1E0',
          100: '#EFDBB1',
          200: '#E5C47F',
          300: '#DCAC4D',
          400: '#D69C2B',
          500: '#D18D0F',
          600: '#CD820A',
          700: '#C87504',
          800: '#C16800',
          900: '#B65200',
        },
      },
      fontFamily: {
        noto: ['Noto Sans JP', { fontFeatureSettings: '"pwid" on' }],
      },
      gridTemplateColumns: {
        auto: 'repeat(auto-fit, minmax(0, 1fr))',
      },
    },
  },
  plugins: [],
}
```
</details>

基本的にはデザイン通りですが，ButtonのHover，Active用の色が少し特殊な色となっており，なぜこの色になるのか理解できませんでした．わかる人がいたら教えてほしいです．

Light Button Normal (`#003EE5`) を 10% 暗く => Light Button Hover (`#0030B2`)
Dark Button Normal (`#7096F8`) を 10% 暗く => Dark Button Hover (`#3F72F6`)

## ボタン

ボタンのデザインは，Hover時やFocus時のデザインが提示されていますが，Focus時のデザインが問題です．Figma上ではFocus時の線をborderによって表現されていますが，最近のブラウザではborderにかぶさるような形でoutlineが表示されます．これはデフォルトでは黒や白なためせっかくのデザインが台無しになってしまいます．今回は，outlineをborderから離して別にデザインしました．

デジタル庁にこのことを問い合わせましたが，2023/03/02現在対応されてません．

また，ダークモードの実装にあたってSecondary Buttonの背景の例が示されていなかったため(正確にはベースの色は設定されているもののその通り実装するとデザインに統一感がないため)，独自に設定しました(`bg-sea-50/10`)．

## ボトムナビゲーション

こちらもダークモード用のアクティブ背景色が示されていなかったため，独自に設定しました(`bg-sea-50/10`)．
また，ダークモード時のバッジの色が少し濃い気がしたので薄くしました(`bg-sun-500/90`)．

## テキスト入力

ラベルにおける「任意」文字やヘルパーテキストの文字色がパレットにない色を使っていますが，統一感を考えてパレット通りにしました．また，outlineはborderから離してます．それら以外はデザイン通りです．

## テキストエリア

デザインに関しては問題ありませんが，文字数制限をする場合についての例が示されています．JavaScriptで文字数を**正確に**数えるのが難しいことや，文字数制限による入力を受け付けない設計はUXを大きく損なうことなどを鑑みて今回は実装しませんでした．簡単な機能ではあるので需要に応じて実装すればいいと思います．

## ラジオボタン

今回は妥協して`accent-color`で済ませました(~~決してめんどくさかったわけではないです~~)．

ラジオボタンのデザインを変えたい場合フルスクラッチで書くのが主流かと思いますが，実は色を変えるだけなら`accent-color`で事足ります．しかし，色が変わるのは選択されているボタンだけで他のボタンは灰色のままです．デザイン案ではエラー時にすべてのボタンの色が赤くなるよう示されており，これを実現できません．
ここで，ラジオボタンがエラーになる状況を考えてみると，必須項目なのに選択されていない場合のみであることがわかります．これは，あらかじめ選択しておく(例えば一番最初にする)ことでそもそもエラーを表示させないことが可能です．

また，ラジオボタンの真ん中の円の大きさがデザイン案では少し小さいですが，円が大きい方が見やすいので妥協してデフォルトのデザインを使います．

タイル形式については，通常版とデータ形式が異なるので補足説明の実装を見送りました．どうしても補足説明を入れたいのであればラジオボタン形式ではなくアコーディオンやカード形式がいいと思います．いずれにせよ情報量が増えるので個人的には入れたくないです．

さらに，デザイン案ではfocus時のデザインが示されておらず，labelへのデザインを独自に設定しました．

## チェックボックス

デザイン案ではfocus時のデザインが示されておらず，labelへのデザインを独自に設定しました．

## モーダルダイアログ

こちらもIEが生きていた時代まではフルスクラッチが主流でしたが，現在ではhtmlがデフォルトで提供する`dialog`を使えます．これは常にトップレイヤーでmodalを表示しつつ背景のbackdropまで設定できるという便利なやつ．
ただ，実装にあたっては１つ罠があって，`dialog`に直接`open`属性を付与するとbackdropが機能しません．要素の`showModal()`や`close()`を実行しないと機能しないのです．なんとめんどくさい ┑(￣Д ￣)┍

Reactではcustom hooksを使ったrefの配信で実装しました．

<details>
    <summary>useDialog.ts</summary>

```ts
import { type Dispatch, type SetStateAction, useCallback, useRef } from 'react'

const useDialog = () => {
  const ref = useRef<HTMLDialogElement>(null)

  const setOpen: Dispatch<SetStateAction<boolean>> = useCallback((setStateAction) => {
    if (ref.current) {
      let open = false

      if (typeof setStateAction === 'boolean') open = setStateAction
      else open = setStateAction(ref.current.open)

      if (open) ref.current.showModal()
      else ref.current.close()
    }
  }, [])

  return { ref, open: ref.current?.open ?? false, setOpen }
}

export default useDialog
```
</details>

`setOpen`の型は`useState`のものをそのままお借りしました．少し危険なかほりがするので素直に`showDialog`と`closeDialog`に分けた方がいいかもしれないです．

また，モーダルを実装するうえで出会う問題として背景のスクロールがあると思います．今回もそれは例外ではなく解決に悩みましたが，以下の記事を参考にして無事解決しました．

https://qiita.com/yowatsuyoengineer/items/b43b64e1419fa285b758#%E8%BF%BD%E8%A8%98

実装内容はとても面白いもので，1pxのずれを無視すればいい感じに使えます．特に[@atomsphere5](https://twitter.com/atomsphere5)様の擬似要素での実装はマークアップに影響を与えないので好きです．今回はこれを採用しました．

## テーブル

テーブル自体のデザインは問題ないのですが，スマホデザイン時のスクロールバーについて説明があったのでここで紹介します．

まずスクロールバーのデザインはFirfoxではできません．ですので特殊な装飾はあきらめましょう(Firfoxを切り捨てるならば問題ないです)．

https://developer.mozilla.org/en-US/docs/Web/CSS/::-webkit-scrollbar#css.selectors.-webkit-scrollbar

そのうえで，現在のモバイルブラウザのスクロールバーの挙動は， **「触らないと見えない」** ようになっています．しかしこれでは，スクロールすべき要素がどこにあるのかわかりません．これを避けるため現在の多くのサイトでは「スクロールできます」という文字だったり，スクロールできることを示す画像が表示されていたりします．
この問題をうまく回避する方法として，強制的にスクロールバーの色を固定してしまう方法があります．

```css
*::-webkit-scrollbar {
  @apply w-3 h-3 rounded-lg bg-sumi-300 dark:bg-sumi-600;
}

*::-webkit-scrollbar-thumb {
  @apply rounded-lg bg-sumi-500 dark:bg-sumi-400;
}
```

これによりスクロールバーが常に表示されます．

## ページネーション

シンプルなデザインで分かりやすいと思いますが，問題があるとすれば矢印ボタンを非表示にするデザインが個人的には好きではないです．このデザインの場合ボタンの位置が現在のページ位置によって異なり，連続クリックできない場合があります．これはとてもUXが悪いです．せめて非活性化するなどレイアウトが変化しないものを推奨したいです．

あと，disabledのときの「なし」はダサいです．

---

以上が「デザインシステム」で示されていたコンポーネントデザインの実装です．一部適当に作ったところもあるので何かあれば修正リクエストお願いします．

最後までお読みいただきありがとうございました．
