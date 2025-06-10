---
title: "React Component Color - Server/Client Components を色分けで可視化するVS Code拡張機能"
emoji: "🎨"
type: "tech"
topics:
  - "react"
  - "typescript"
  - "contest2025ts"
published: true
---

## はじめに

モダンなReact開発において、Server ComponentsとClient Componentsの境界を意識した開発が重要になってきました。しかし、コードを書いている最中に「このコンポーネントはサーバーサイドで実行されるのか、クライアントサイドで実行されるのか」を瞬時に判断するのは難しく、開発者の悩みの種となっています。

この問題に対するアプローチとして、[SupremeDeity](https://github.com/SupremeDeity)さんが「[react-component-colors](https://github.com/SupremeDeity/react-component-colors)」を作成していました。このVS Code拡張機能は、ReactのServer ComponentsとClient Componentsを色分けして可視化することで、開発者がコンポーネントの種類を一目で理解できるようにします。この拡張機能はとても便利なのですが、キャッシュされておらずコードを編集するたびに色が元に戻ったり、Client Component内のServer ComponentはすべてClient Componentとして扱われるなど、いくつかの制限がありました。また、しばらく更新が止まっているようです。

そこで、私が「react-component-color」という名前で同様の機能を持つVS Code拡張機能を作成しました。

https://marketplace.visualstudio.com/items?itemName=kage1020.react-component-color

また、[@makotot](https://zenn.dev/makotot)さんがちょうど時を同じくして「[Next.js App Routerのサーバー・クライアントコンポーネントの境界をVS Code上で可視化する](https://zenn.dev/makotot/articles/ec6c631bc83e9e)」という記事で、アイコン表示によってServer ComponentsとClient Componentsの境界を可視化する拡張機能を公開されました。こちらも非常に便利で、私の拡張機能と併用することができます。Server ComponentとClient Componentの話はこちらに詳しく書かれていますので、ぜひご覧ください。

## 拡張機能の概要

「React Component Color」は、エディタ上でReactコンポーネントを視覚的に色分けし、Server ComponentsとClient Componentsを即座に識別できるようにする拡張機能です。

### 主な特徴

- **🎨 視覚的なコンポーネント識別**: JSX/TSXコンポーネントを自動検出し、色でコード分け
- **🔍 スマートな検出**: `'use client'`ディレクティブを解析してServer/Client Componentsを識別
- **⚙️ 高度なカスタマイズ**: 背景色、境界線、下線、テキスト色を個別に設定可能
- **🔄 リアルタイム更新**: コード編集に応じて即座に色分けを更新
- **📁 インポート解析**: インポートチェーンを追跡し、TypeScriptのパスマッピングもサポート
- **💾 パフォーマンス最適化**: キャッシュシステムで高速解析

## なぜこの拡張機能が必要なのか

### Server/Client Components の課題

React Server Componentsが導入されて以降、開発者は次のような課題に直面しています：

1. **実行環境の判別困難**: コンポーネントがサーバーとクライアントのどちらで実行されるかを瞬時に判断するのが困難
2. **遅い気づき**: 不適切なAPIの使用により、ビルド時や実行時にエラーが発生して初めて問題に気づく
3. **バンドルサイズの予期しない増加**: Client Componentの範囲を把握できず、バンドルサイズが意図せず膨らむ
4. **セキュリティリスク**: サーバー専用の処理がクライアントサイドに露出するリスク

### よくある開発シーン

以下のようなエラーメッセージを見たことがある開発者は多いのではないでしょうか：

```
Error: You're importing a component that needs `useState`.
This React hook only works in a client component.
To fix, mark the file (or its parent) with the `"use client"` directive.
```

このような問題を**実装段階で事前に防ぐ**ことが、この拡張機能の主要な目的です。

## 機能詳細

### コンポーネントの分類と色分け

拡張機能は以下のルールでコンポーネントを分析・色分けします：

- **Server Components**: `'use client'`ディレクティブのないコンポーネント

![](/images/react-component-color/server-component.png)

- **Client Components**: `'use client'`ディレクティブを持つファイルのコンポーネント

![](/images/react-component-color/client-component.png)

### インポート解析の仕組み

拡張機能は以下の方法でコンポーネントの種別を正確に判定します：

1. **相対インポートの追跡**: `./Component`形式のインポートを解析
2. **TypeScriptパスマッピング**: `@/components/Button`などの設定を考慮
3. **インデックスファイル解決**: `index.js`や`index.ts`を適切に処理
4. **各種インポートパターン**: 名前付き、デフォルト、名前空間インポートに対応

### カスタマイズ可能な設定

開発者の好みや開発環境に応じて、細かい設定が可能です：

```json
{
  "reactComponentColor.enable": true,
  "reactComponentColor.serverComponent.backgroundColor": "#2E3440",
  "reactComponentColor.serverComponent.borderColor": "#4C566A",
  "reactComponentColor.serverComponent.underlineColor": "#4C566A",
  "reactComponentColor.serverComponent.textColor": "#4EC9B0",
  "reactComponentColor.clientComponent.backgroundColor": "#FFFFFF",
  "reactComponentColor.clientComponent.borderColor": "#FFB8D1",
  "reactComponentColor.clientComponent.underlineColor": "#FFB8D1",
  "reactComponentColor.clientComponent.textColor": "#FF719B"
}
```

各コンポーネントタイプに対して以下の4つの色を個別に設定できます：
- `backgroundColor`: 背景のハイライト色
- `borderColor`: コンポーネント周りの境界線色
- `underlineColor`: 下線色
- `textColor`: コンポーネント名のテキスト色

## 技術的な実装について

makototさんと同様に、開発はGitHub CopilotのAgent ModeとClaude Sonnet 4を活用して行いました。[CONVERSATION.md](https://github.com/kage1020/react-component-color/blob/main/CONVERSATION.md)に、開発中のやり取りを記録しています。拡張機能のアイコンはChatGPTに生成してもらいました。

### パフォーマンス最適化

大規模なプロジェクトでも快適に使用できるよう、以下の最適化を実装しています：

1. **キャッシュ**: 一度解析した結果をキャッシュし、変更があった部分のみ再解析
2. **効率的なAST解析**: TypeScript Compiler APIを使用した最適化されたパース処理
3. **最小限のVS Code影響**: エディタのパフォーマンスに与える影響を最小限に抑制

### 対応ファイル形式

以下のファイル形式をサポートしています：
- `.jsx` - JavaScript with JSX
- `.tsx` - TypeScript with JSX
- `.js` - JavaScript（JSXを含む場合）
- `.ts` - TypeScript（JSXを含む場合）

## おわりに

この拡張機能により、React開発における Server/Client Components の境界をより直感的に把握できるようになります。コードを書いている段階で実行環境を意識できるため、開発効率の向上とバグの早期発見に貢献できると考えています。

機能の改善提案やバグ報告は、[GitHubリポジトリ](https://github.com/kage1020/react-component-color)でお待ちしています。

モダンなReact開発をより快適に進めるため、ぜひこの拡張機能をお試しください！🚀
