---
title: "eslint-plugin-importにおいてcssをimportするときの話"
emoji: "👋"
type: "tech"
topics:
  - "css"
  - "react"
  - "typescript"
  - "eslint"
published: true
published_at: "2022-10-24 00:55"
---

ESLintプラグインの1つにimportの順番を整理するというものがある．

https://github.com/import-js/eslint-plugin-import

これを使うと，ごちゃごちゃなimportをアルファベット順やパッケージごとに整理できる．

```tsx:ごちゃごちゃ
import '@styles/global.css'
import { getAData, getBData, BDataType, getCData } from '@libs/tools'
import Image from 'next/image'
import type { DataType } from '@types/libs'
import React from 'react'
```

```tsx:スッキリ！
import React from 'react'
import Image from 'next/image'

import { getAData, getBData, getCData } from '@libs/tools'

import '@styles/global.css'  // (*)

import type { BDataType } from '@libs/tools'
import type { DataType } from '@types/libs'
```

# unassigned importの問題

ただし，これには1つ欠点(?)がある．それは上記のcssのような，代入せずに直接importするものに対してはlinterが働かないことである．上の例では型定義の前においているが，何もしないと`get*Data`の前に来る．tailwind cssなどを使っていると`globals.css`はmoduleではなくcssファイルとして読み込みたいため安易に代入できない．

# なぜ適用されない？

Issueにて開発者が採用しない理由を述べていた．

https://github.com/import-js/eslint-plugin-import/issues/1084

要するに，直接importすることによる副作用をlinterが無視するべきではないということ．
今回はcssが1つであるから特に考慮する必要はないが，もし複数読み込む必要がある場合，cssの読み込み順序をアルファベット順などで書き換えてしまうと正しい見た目にならない可能性がある．
開発者はこのような副作用を気にしてあえて実装していないようだ．もちろんオプションとして切り出す可能性も議論されているが，消極的なようだ．

# 解決策なし

現状，調べた限りでは解決策は見当たらなかった．linterが労力を割いて副作用を検証することは難しいだろうし，今回はcssしか取り上げなかったが，npm modulesに対応していないパッケージなども同じ状況になりうるので人の手でやるしかないように思う．

また，これを解決する提案の1つとしてIssueに上がっていたのは`pathGroups`に新しいオプションを設定すること．

https://github.com/import-js/eslint-plugin-import/issues/970#issuecomment-662943362

```json
{
  "pathGroups": [
    {
      "pattern": "{.,..}/**/*.+(css|sass|less|scss|pcss|styl)",
      "unnamed": true, // this is a new option
      "group": "unknown",
      "position": "after"
    }
  ]
}
```

しかし，2020年7月で議論が終わっているので今すぐどうにかなるものでもなさそう．

---

探せばうまくやってくれるプラグインなどありそうだが，それを探す労力と利便性が釣り合わないので今回はここまで．誰かいいのを見つけたら教えてください．
