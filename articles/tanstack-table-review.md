---
title: "TanStack/tableを使ってみた感想"
emoji: "👻"
type: "tech"
topics:
  - "react"
  - "headlessui"
  - "tanstack"
published: true
published_at: "2022-07-03 15:10"
---

先日2022/07/01にメジャーアップデートがあったTanStack/tableが便利そうだったので，その感想をここに置いておきます．

# TanStack/table とは

> Headless UI for building powerful tables & datagrids for React, Solid, Vue, Svelte and TS/JS.
> https://github.com/TanStack/table

いわゆるHeadless UIを提供するライブラリであり，テーブルのソートやページネーションなどの面倒な機能を代わりにやってくれる．

# 生まれ変わったreact-table

メジャーアップデートが来る前(v7)までは`react-table`という名前で使われていた．機能面では以前のバージョンとあまり変わらないが大きく変わったのは以下の2つ．

* TypeScriptフルサポート
* React以外にVue, Svelte, Solidのメジャーフレームワークに対応

これまでTypeScriptのサポートがコミュニティに投げられていたことから敬遠していた人も多いだろう．

# 使い方

:::message
以降のコードはReactで書いています．
:::

Headless UIなのでマークアップは書かなければいけない．TanStack/tableはそのためのロジックを以下のように提供する．

```jsx
const table = useReactTable({
  data: data, // json形式
  columns: columns, // header
  getCoreRowModel: getCoreRowModel() // おまじない
});
```

マークアップは`map`をひたすら回すだけでいい．

```jsx
<table>
  <thead>
    {table.getHeaderGroups().map(headers => (
      <tr key={headers.id}>
    {headers.headers.map(header => (
      <th key={header.id} className={styles.th}>
        {header.isPlaceholder
        ? null
        : flexRender(
        header.column.columnDef.header,
        header.getContext()
          )}
      </th>
    ))}
      </tr>
    ))}
  </thead>
  <tbody>
    {table.getRowModel().rows.map(row => (
      <tr key={row.id}>
    {row.getVisibleCells().map(cell => (
      <td key={cell.id} className={styles.td}>
        {flexRender(cell.column.columnDef.cell, cell.getContext())}
      </td>
    ))}
      </tr>
    ))}
  </tbody>
</table>
```

# Pros & Cons & Thoughts

このライブラリのいいところは，あらかじめヘッダ情報を渡して整形してもらうことで，マークアップがデータ構造に影響しにくいこと(特に複雑なヘッダに対して)．
また，前のバージョンに比べて必要な関数などが減りより直感的に操作しやすくなった．
もちろんソートなどのときにstate管理してくれるので自前でロジックを用意する必要がありません．

一方，コードを見ればわかりますが，mapの回し方が少し冗長です．特にcellの出力に`flexRender(cell.column.columnDef.cell, cell.getContext())`と書かなければいけないのは厄介です．まだドキュメントが整備されていないためこのコードの必要性を理解していないですが，フィルタなどに使うためのwrapperなんじゃないかとにらんでます．
また，メソッド名がわかりづらくパッケージにもコメントがないためどう使えばいいかわからないのが現状ですね．これは開発の進展を待つしかないです．

OSSなので参加してみたいのですが時間が．．．(言い訳)

機能としてはフィルタリング，ソート，ページネーション，MUI対応等申し分ない機能が盛りだくさんで便利です．特にフィルターとページネーションの組み合わせを自前でやろうとするとコード量が莫大になるのでそれができるのは控えめに言って神です．

---

以上，TanStack/tableを使って出た小言でした．

---

2022/09/04 一部誤字訂正
