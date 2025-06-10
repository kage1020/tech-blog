---
title: "Javascriptの論理演算をまとめてみた"
emoji: "📔"
type: "tech"
topics:
  - "javascript"
  - "react"
published: true
published_at: "2022-07-24 11:07"
---

最近Reactを書いていてJavascriptの論理演算の理解が甘いことに気がついたので，MDNドキュメントを分かりやすく(？)書き換えながら論理演算の仕様と例を並べときます．

日本語対応している素晴らしい本家様はこちらです．
https://developer.mozilla.org/ja/docs/Web/JavaScript

## 列挙の前に

以降のコードはES2015(ES6)を基準にしていますが，一部ES2020が出てきます．
また，筆者のReact好きがにじみ出てjsxを書くことがありますが，本記事ではjsxについて詳しく説明しませんのでご了承ください．

コード例とその出力は以下のように表します．

```javascript
console.log("Hello World!");     // 実行コード
// "Hello World!"                // 出力
```

# Truthy, Falsy

Javascriptには様々な型がありますが，その値によって真偽値は異なります．その中でも以下に挙げるものは特別に偽(`false`)をとるとしてFalsyと呼ばれます．

| `false`     | description    |
| ----------- | -------------- |
| `0`, `-0`   | 数字(number)   |
| `0n`        | 数字(BigInt)   |
| `""`        | 空文字         |
| `null`      | 値が存在しない |
| `undefined` | 未定義         |
| `NaN`       | 数字でない     |
| `Infinity`  | 無限大         |

これら以外はTruthyと呼ばれ，条件分岐では常に`true`を返します．特に，以下の値はFalsyと対比関係にあるため覚えておいた方がいいです．

| `true` | description    |
| ------ | -------------- |
| `1`    | 数字           |
| `foo`  | 任意文字列     |
| `[]`   | 空配列         |
| `{}`   | 空オブジェクト |

しかし，後述しますが`true`と**厳密でない**等価比較を行った際以下は`false`になります．

| `false`(一例) | description                           |
| ------------- | ------------------------------------- |
| `[]`          | 空配列(`Array(0) == 1`と等価)          |
| `Object(0)`   | 空オブジェクト(`Number(0) == 1`と等価) |
| `"0"`         | 0の文字(`0 == 1`と等価)                |

おまけとして，`{} == true`と`{} == false`は常にエラーを返します．これは`{}`自体がオブジェクト初期化子という演算子のため，演算子が2回続き構文エラーとなるためです．

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Object_initializer

**2022/11/26 追記**
また，常にFalsy(`undefined`)を返すオブジェクトとして`document.all`があります．今は亡きIEをはじき出すために使われたらしいです．

https://qiita.com/suin/items/461c096bef318a259c80#javascript%E3%81%AB%E3%81%AFfalsy%E3%81%AA%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%8C%E3%81%82%E3%82%8B

# 基本演算

条件分岐等の論理演算ではあまり使われませんが，コードのスリム化をする際に使う人がいるかもしれないので書いときます．

## 加算`+`

```javascript
console.log(true + true + false);
// 2
```

## 減算`-`

```javascript
console.log(false - true);
// -1
```

## 乗算`*`

```javascript
console.log(2 * true + 4 * false);
// 2
```

## 除算`/`

```javascript
console.log(true / 2 + false / 4);
// 0.5
```

## 剰余`%`

```javascript
console.log((true + true + false) % 2);
// 0
```

## 累乗`**`

```javascript
console.log((true + false + true) ** 3);
// 8
```

# ブール演算(論理演算)

## 論理積`&&`

左から各オペランド(変数)を検査していき，Falsyならそのオペランドを返し，全てTruthyなら最終オペランドを返します．

```javascript
console.log(1 && null && "foo");
// null
console.log(true && "bar");
// "bar"
```

Reactではネストが深くなるのを防ぐためによく使います．

```jsx:React
<div>
  {/* ログインして，かつ名前を取得しているとき */}
  {isLogined && name && <span>Your name is {name}!</span>}
</div>
```

## 論理和`||`

左から各オペランドを検査していき，Truthyならそのオペランドを返し，全てFalsyなら最終オペランドを返します．

```javascript
cosnole.log(null || 1 || "");
// 1
console.log(null || undefined || "0");
// "0"
```

## null合体`??`

左から各オペランドを検査していき，`null`または`undefined`なら右のオペランドを返し，それ以外はそのオペランドを返します．これはES2020で追加された項目です．
用途としては，APIで非同期取得した値を表示するときの初期化として使えます．

```javascript
console.log(null ?? undefined ?? 1);
// 1

const someFunc = (id) => {
  const key = id ?? 0; // 引数の初期化としても使える
  ...
}
```

```jsx:React
const name = await getName('https://exmaple.com?id=1')

// fetchが完了するまでは'missing'が表示される
return <span>Your Name is {name ?? 'missing'}!</span>
```

## 否定`!`

オペランドをboolに変換したうえで反転した値を返します．変換はTruthyかFalsyかによって変換されます．

```javascript
console.log(!"0");
// false
console.log(![]);
// false
```

TruthyやFalsyをデベロッパーが区別することは容易ではありません．そこで二重否定`!!`を用いることで明示的にboolに変換することができます．

```javascript
console.log(!!0);
// false
console.log(!!"0");
// true
```

## 等価`==`

比較するオペランドが違う型の場合，数字(1)に変換して比較しようとします．`"1"`や`[0]`は数字に変換できますが，`"foo"`は変換できません．また，`null`と`undefined`を区別しません．

```javascript
console.log("0" == false);
// true
console.log([] == false);
// true
console.log(Object(0) == false);
// true
console.log(null == undefined);
// true
```

`[]`や`Object(0)`はTruthyですが，比較演算では1に変換できないので0(`false`)です．

### 参考

`if`文に値そのものを渡したとき，JavascriptではTruthyかFalsyかの判定をします．

```javascript
if ([]) console.log("Truthy");
else console.log("Falsy");
// "Truthy"
```

## 不等価`!=`

等価`==`の否定です．

```javascript
console.log(1 == "1");
// true
console.log(1 != "1");
// false
```

## 厳密等価`===`

等価`==`と違い，型の変換をせずに比較します．つまり，型の比較をします．また，`null`と`undefined`を区別します．

```javascript
console.log("1" == true);
 // false
 console.log("" + [0] === "0");
 // true
 console.log(null === undefined);
 // false
```

## 厳密不等価`!==`

厳密等価`===`の否定です．

```javascript
console.log(1 === "1");
// false
console.log(1 !== "1");
// true
```

## 大なり`>`

右のオペランドよりも左のオペランドの方が大きい場合`true`を返し，それ以外は`false`を返します．

```javascript
console.log("3" > 1);
// true
console.log(["b"] > ["a"]);
// true                      // 文字コードがbの方が大きいため
console.log({ "a": 3 } > { "a": 1 });
// false
```

オペランドの比較は[抽象関係比較アルゴリズム](https://tc39.es/ecma262/#sec-abstract-relational-comparison)により以下のコード(ざっくり)のように判定されます．

:::details 比較コード

```javascript:javascript like
 // xに第一オペランド，yに第二オペランド
const isLessThan = (x, y) => {
  const px = toPrimitive(x, number); // オブジェクトを数字に変換
  const py = toPrimitive(y, number); // オブジェクトを数字に変換
  if (typeof px === 'string' && typeof py === 'string') {
    const lx = length(px); // pxの「長さ」を取得
    const ly = length(py); // pyの「長さ」を取得
    for (const i = 0; i < min(lx, ly); l++) {
      const cx = px[i];
      const cy = py[i];
      if (cx < cy) return true;
      if (cx > cy) return false;
    }
    if (lx < ly) return true;
    return false;
  }
  else {
    if (typeof px === 'bigint' && typeof py === 'string') {
      const ny = BigInt(py);
      if (ny === undefined) return undefined;
      return px < ny;
    }
    if (typeof px === 'string' && typeof py === 'bigint') {
      const nx = BigInt(px);
      if (nx === undefined) return undefined;
      return nx < py;
    }
    const nx = Number(px);
    const ny = Number(py);
    if (typeof nx === typeof ny) return nx < ny;
    if (nx === NaN || ny === NaN) return undefined;
    if (nx === -Infinity || ny === Infinity) return true;
    if (nx === Infinity || ny === -Infinity) return false;
    return nx < ny;
  }
}
```

:::

肝は`toPrimitive`関数で，ここで数字に変換できれば比較結果が返され，変換できなければ`NaN`がコードを下っていきます．`nx === ny === NaN`となったまま最後の比較に到達し，`if (typeof nx === typeof ny)`で`nx < ny`，すなわち`false`を返します．

## 小なり`<`

大なり`>`の否定です．

```javascript
console.log("3" < 1);
// false
console.log(["b"] < ["a"]);
// false                      // 文字コードがbの方が大きいため
console.log({ "a": 3 } < { "a": 1 });
// false
```

## 以上`>=`

大なり`>`の`if (typeof nx === typeof ny) return nx <= ny`で`true`を返します．

```javascript
console.log(undefined >= 0);
// false
console.log({ "a": 3 } >= { "a": 1 });
// true
```

## 以下`<=`

以上`>=`の否定です．

```javascript
console.log(undefined <= 0);
// false
console.log({ "a": 3 } <= { "a": 1 });
// true
```

# ビット演算

`true`や`false`はそれぞれ`1`，`0`と互換性(？)があるのでビット演算でも条件分岐に使えます．
値を2ビット整数で表す場合は以下のように2の補数を整数部と符号部を分けて記述します．

```javascript
7 = 0 0111
-7 = 1 1001
```

## 論理積`&`

符号付32ビット整数に変換後，各ビットでANDをとって10進数を返します．

```javascript
console.log(13 & 7); // 1101  &  0000 0111 = 0101
// 5
console.log([true, true, true].reduce((acc, cur) => acc & cur, 1);
// 1
```

## 論理和`|`

符号付32ビット整数に変換後，各ビットでORをとって10進数を返します．

```javascript
console.log(13 | 7); // 1101  &  0111 = 1111
// 15
console.log([true, false, false].reduce((acc, cur) => acc | cur, 0);
// 1
```

## 否定`~`

符号付32ビット整数に変換後，ビット反転して10進数を返します．

```javascript
console.log(~13); // ~(0 1101) -> 1 0010 -> 1 0001 -> -(0 1110) -> -14
// -14
console.log(-13 & ~7); // 1 10000 & 1 11000 = 1 10000
// -16
```

1例目はビット反転後，10進数に変換するために1を引いて再度ビット反転しています．
2例目は2の補数の-13とビット反転した7の論理積をそのまま返しています．

## 排他的論理和`^`

2値のうち片方のみが1のときのみ1がたちます．

```javascript
const isGrowing = (downward, upward) => downward ^ upward; // 階段の照明
isGrowing(true, true);
// 0
isGrowing(true, false);
// 1
```

## 右シフト`>>`

ビット列の左端のコピーを左隣から押し込みます．右端のあふれたビットは無視されます．要するに位が下がるので2で割ることとほぼ同義です(余りは切り捨てられますが)．
ちなみに「2で割る」は2進数においての話で，10進数でシフトをしようとすると「10で割る」ことを意味します．

```javascript
console.log(1024 >> 1 === 1024 / 2);
// true
console.log(-1024 >> 1);
// -512
```

Javascriptは符号ビットを演算対象にしないので負の数を右シフトしても絶対値は大きくなりません．

## 左シフト`<<`

**2022/07/27 修正**
~~ビット列の左端のコピーを左隣から押し込みます．~~ 0を右端から押し込みます．左端のあふれたビットは無視されます．要するに位が上がるので2を掛けることと同義です．

```javascript
console.log(1024 << 1 === 1024 * 2);
// true
console.log(2 ** 31 << 1);
// 0
```

Javascriptはビット列を32ビットで表しそのうち最上位を符号として使うため，31ビットを超える数字は表現できません．ただし，表現できないのはビット整数であって，数字はさらに大きな値を表すことができます．

```javascript
console.log(2 ** 64);
// 18446744073709552000
```

## 符号なし右シフト`>>>`

右シフト`>>`は左端ビットを複製しますが，符号なしでは常に正の数にするために0を挿入します．

```javascript
console.log(-1024 >> 23); // -1024 = 1111 1111 1111 1111 1111 1011 1111 1110
// 511                    // 511   = 0000 0000 0000 0000 0000 0001 1111 1111
```

## 符号なし左シフト`<<<`

存在しません．左シフトはそのオペランドによって最上位ビットの値が変わるので符号が常に正とはならないためです．

# おまけ

Reactでよく使う論理演算を載せておきます．

## 3項演算子(条件演算子)`?:`

可読性が問題の賛否両論ある演算子です．第1オペランドを評価して，`true`なら第2オペランドを，`false`なら第3オペランドを返します．
筆者としては，ネストしなければワンライナーでかけて美しいと思うのですが，手続き型を扱っている人ならではの視点なんですかね？

```javascript
console.log("1" == 1 ? "Welcome!" : "Hello World!");
// "Welcome!"
```

私はReactでログイン状態によって表示を切り替えたい，かつオペランドが1行で済むときに良く使います．

```jsx:React
<div>
  {isLoggined ? <span>Welcome!</span> : <span>Hello World!</span>}
</div>
```

オペランドが複数行になったら可読性が下がるので迷わず論理積`&&`を使いましょう．

```jsx:React
<div>
  {isLoggined && (
    <>
      <span>Welcome!</span>
      <span>ようこそ！</dpan>
    </>
  )}
  {!isLoggined && (
    <>
      <span>Hello World!</span>
      <span>こんにちは世界！</span>
    </>
  )}
</div>
```

## オプショナルチェーン`?.`

ES2020で追加された演算子です．オブジェクトのネストしたキーを参照するときに存在しないキーの属性を参照しようとしたとき，エラーを吐かずに`undefined`を返してくれます．

```javascript
const obj = { "a": 1, "b": {"c": 2} };
console.log(obj.d);
// undefined
console.log(obj.d.e);
// Uncaught TypeError: Cannot read properties of undefined (reading 'e')
console.log(obj.d?.e);
// undefined
```

ReactではAPIなどの非同期処理時に，オブジェクトの値を取得しているかどうかによる表示の出し分けが簡単になります．

```jsx:React
// { name: {"firstName": string, "lastName": string} }が返ってくる
const profile = await getProfile('https://exmaple.com?id=1')

return <span>Your Name is {profile?.name?.firstName ?? 'missing'}!</span>
```

---

以上，Javascriptにおける論理演算(+$\alpha$)でした．

# 参考文献

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/

https://tc39.es/ecma262/#sec-abstract-relational-comparison
