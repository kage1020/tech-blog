---
title: "Google Ads Query Languageを型安全に書く"
emoji: "🔍"
type: "tech"
topics: ["typescript", "vscode", "googleads", "oss"]
published: true
---

## はじめに

Google Ads APIを使った広告運用の自動化やレポート作成では、**GAQL (Google Ads Query Language)** というSQL風のクエリ言語を使ってデータを取得します。

```typescript
const query = `
  SELECT
    campaign.id,
    campaign.name,
    metrics.impressions,
    metrics.clicks
  FROM campaign
  WHERE campaign.status = 'ENABLED'
`;
```

しかし、GAQLには以下のような課題がありました:

- ❌ **補完が効かない**: リソース名やフィールド名を手入力する必要がある
- ❌ **バリデーションがない**: 実行するまでエラーに気づけない
- ❌ **typoに弱い**: `campain`と書いても気づきにくい
- ❌ **ドキュメント参照が面倒**: 173種類のリソースと数千のフィールドを覚えられない

これらの課題を解決するため、**GAQL用のTypeScriptツールキット**を1から実装しました。

https://github.com/kage1020/google-ads-query-language

## 作ったもの

モノレポで3つのパッケージを開発しました:

### 1. [@gaql/core](https://www.npmjs.com/package/@gaql/core)
バリデーション、スキーマ管理、パーサーなどのコア機能を提供するライブラリ

```typescript
import { validateQuery } from '@gaql/core';

const result = validateQuery(query);
if (!result.valid) {
  console.error(result.errors);
}
```

### 2. [@gaql/cli](https://www.npmjs.com/package/@gaql/cli)
ファイルやstdinからクエリを検証するCLIツール

```bash
# ファイルを検証
gaql validate query.ts

# JSON出力でCI/CDに組み込み可能
gaql validate query.ts --format json
```

### 3. [gaql-vscode](https://marketplace.visualstudio.com/items?itemName=kage1020.gaql-vscode)
リアルタイムバリデーション、補完、IntelliSenseを提供するVS Code拡張

![VSCode拡張のデモ: GAQLクエリのリアルタイムバリデーションと補完の様子](/images/google-ads-query-language/demo.gif)

## 主な機能

### バリデーション

テンプレートリテラル内のGAQLクエリをリアルタイムで検証します。例えば、存在しないフィールドを指定すると:

![バリデーションエラーの例: 存在しないフィールドが波線で示されている様子](/images/google-ads-query-language/validation-error.png)

エラー箇所に波線が表示されます。

```
% gaql validate test.ts
✔ Read file: test.ts

=== GAQL Validation Results ===

Total queries: 1
✓ Valid: 0
✗ Invalid: 1

✗ Query at line 2: Invalid
  Query:
    SELECT
        campaign.campain_name,
        metrics.clicks
      FROM campaign

  Errors:
    • Invalid field: campaign.campain_name for resource campaign
      Suggestion: campaign.campaign_budget
```

CLIでは`campaign.campaign_budget`のような修正候補を提案します。実際の開発では、Google Ads APIを叩いてエラーが返ってきてから気づくことが多かったので、コーディング中に気づけるのは便利です。

### 補完

![補完の例 SELECT句でフィールド名の候補が表示されている様子](/images/google-ads-query-language/completion.png)

`SELECT cam`と入力すると、そのリソースで使えるフィールドの候補が表示されます。campaignリソースなら`id`, `name`, `status`など50以上のフィールドがあり、全て覚えるのは無理なので、補完で探せるのは助かります。

また、`FROM`の後では173種類のリソース名が候補に出るため、「あのリソース名なんだっけ」と公式ドキュメントを開く回数が減りました。

### ホバー情報

![ホバー情報の例: フィールド名にカーソルを合わせると型と説明が表示される様子](/images/google-ads-query-language/hover.png)

フィールド名にカーソルを合わせると、型と説明が表示されます。Google Ads APIのフィールドは名前だけでは用途が分かりにくいものも多く、いちいちドキュメントを確認していたので、エディタ内で完結できるのは地味に便利です。

### APIバージョンの切り替え

Google Ads APIはv19、v20、v21が現在サポートされており、バージョンごとに使えるフィールドが異なります。VSCodeの設定で切り替えると、そのバージョンのスキーマでバリデーションされます。

```json
{
  "gaql.apiVersion": "21"
}
```

CLIでは`--api-version`または`-a`で指定できます。

```bash
gaql validate query.ts --a 20
```

### アクティベーション制御

VSCodeでは状況によってバリデーションをかけたくない場合もあると思うので、デフォルトではバリデーションは無効です。`// @gaql`コメントを書いた箇所だけ有効になります。

```typescript
// バリデーション有効
// @gaql
const query1 = `SELECT campaign.id FROM campaign`;

// 何も書かなければ無効（文字列として扱われる）
const query2 = `This won't be validated`;

// 一時的に無効化
// @gaql-disable-next-line
const rawQuery = `SELECT * FROM campaign`;  // GAQLでは使えない構文だが無視される
```

全ファイルで常に有効にしたい場合は、設定で`"gaql.activationMode": "always"`にすることもできます。

## 使い方

### VSCode拡張

1. マーケットプレイスから「Google Ads Query Language」をインストール
2. TypeScript/JavaScriptファイルでGAQLクエリを書く
3. コメント `// @gaql` でバリデーション有効化

```typescript
// @gaql
const query = `
  SELECT
    campaign.id,
    campaign.name,
    metrics.clicks
  FROM campaign
`;
```

リアルタイムでエラーチェック、補完、型情報が表示されます。

### CLI

```bash
# インストール
npm install -g @gaql/cli

# ファイルを検証
gaql validate query.ts

# パイプで使用
cat query.ts | gaql validate

# APIバージョンを指定
gaql validate query.ts --api-version 21
```

#### 出力形式

CLIは4種類の出力形式をサポートしています。`--format`オプションで指定できます。

##### 1. text（デフォルト）

人間が読みやすいシンプルなテキスト形式。無効なクエリのみを表示します。

```bash
gaql validate query.ts
# または明示的に
gaql validate query.ts --format text
```

```
=== GAQL Validation Results ===

Total queries: 2
✓ Valid: 1
✗ Invalid: 1

✗ Query at line 8: Invalid
  Query:
    SELECT
      campaign.invalid_field,
      metrics.clicks
    FROM campaign

  Errors:
    • Invalid field: campaign.invalid_field for resource campaign
      Suggestion: campaign.id
```

##### 2. json

機械処理向けのJSON形式。CI/CDパイプラインでの利用に便利です。

```bash
gaql validate query.ts --format json
```

```json
{
  "totalQueries": 2,
  "validQueries": 1,
  "invalidQueries": 1,
  "results": [
    {
      "query": "SELECT campaign.invalid_field FROM campaign",
      "line": 8,
      "valid": false,
      "errors": [
        {
          "type": "invalid_field",
          "message": "Invalid field: campaign.invalid_field for resource campaign",
          "line": 2,
          "column": 5,
          "length": 23,
          "field": "campaign.invalid_field",
          "suggestion": "campaign.id"
        }
      ]
    }
  ]
}
```

GitHub Actionsでの使用例:

```yaml
- name: Validate GAQL queries
  run: |
    npm install -g @gaql/cli
    gaql validate src/**/*.ts --format json > validation-results.json

- name: Check validation results
  run: |
    if [ $(jq '.invalidQueries' validation-results.json) -gt 0 ]; then
      echo "Invalid queries found"
      exit 1
    fi
```

##### 3. llm

LLMが解析しやすい1行形式のエラー出力。Claude CodeなどのAIツールと組み合わせる際に便利です。

```bash
gaql validate query.ts --format llm
```

```
SUMMARY: Total=2, Valid=1, Invalid=1

[ERROR] Line 9 (col 5-28): invalid_field - Invalid field: campaign.invalid_field for resource campaign | Suggestion: campaign.id | Field: campaign.invalid_field | Query: SELECT campaign.invalid_field, metrics.clicks FROM campaign
```

##### 4. rich

テーブルとボックスを使った視覚的にリッチな表示。ターミナルでの可読性が高いです。

```bash
gaql validate query.ts --format rich
```

```
┌─────────────────────────────────────────────────┐
│              📊 Summary                         │
│                                                 │
│  GAQL Validation Results                        │
│                                                 │
│  Total queries: 2                               │
│  ✓ Valid: 1                                     │
│  ✗ Invalid: 1                                   │
└─────────────────────────────────────────────────┘

┌──────────────────┬──────────────────────────────────────────┐
│ Property         │ Value                                    │
├──────────────────┼──────────────────────────────────────────┤
│ Line             │ 9                                        │
│ Query            │ SELECT campaign.invalid_field, ...       │
│ Error 1          │ Invalid field: campaign.invalid_field... │
│ Type             │ invalid_field                            │
│ Position         │ Line 2, Column 5-28                      │
│ Field            │ campaign.invalid_field                   │
└──────────────────┴──────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────┐
  │           💡 Did you mean?                      │
  │                                                 │
  │  SELECT campaign.id, metrics.clicks             │
  │  FROM campaign                                  │
  └─────────────────────────────────────────────────┘
```

### プログラム利用

```typescript
import { validateQuery, getCompletions } from '@gaql/core';

// バリデーション
const result = validateQuery(query);

// 補完候補取得
const completions = getCompletions(query, cursorPosition);

// スキーマクエリ
import { getResourceNames, getFieldsForResource } from '@gaql/core';
const resources = getResourceNames();
const fields = getFieldsForResource('campaign');
```

## おわりに

Google Ads Query Languageを型安全に書けるツールキットを、コアライブラリ・CLI・VSCode拡張の3つのパッケージとして実装しました。

これにより、GAQL開発の課題だった「補完が効かない」「バリデーションがない」「typoに弱い」といった問題を解決し、**型安全で快適な開発体験**を実現できました。

Google Ads APIを使った開発をされている方、これから始める方のお役に立てば幸いです。

このプロジェクトはコミュニティライブラリ[google-ads-api](https://github.com/Opteo/google-ads-api)パッケージの型定義を活用して実現できました。高品質なTypeScript型定義を提供・メンテナンスしていただいている全てのコントリビューターの皆様に深く感謝いたします。

## リンク

- **GitHub**: https://github.com/kage1020/google-ads-query-language
- **npm (@gaql/core)**: https://www.npmjs.com/package/@gaql/core
- **npm (@gaql/cli)**: https://www.npmjs.com/package/@gaql/cli
- **VSCode Marketplace**: https://marketplace.visualstudio.com/items?itemName=kage1020.gaql-vscode
- **google-ads-api**: https://github.com/Opteo/google-ads-api
