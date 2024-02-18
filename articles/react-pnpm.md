---
title: "React+TypeScript環境構築(pnpm用)"
emoji: "🐙"
type: "tech"
topics:
  - "react"
  - "typescript"
  - "pnpm"
published: true
published_at: "2023-02-22 02:12"
---

`create-react-app`でTypeScriptアプリを作るときnpm，yarnでは問題ないがpnpmだとうまくいかない．2023/02/22現在のインストルー方法をメモしときます．

リポジトリはこちらから

https://github.com/kage1020/ReactTypeScriptApp

# `create-react-app`

とりあえずreactアプリを作る

```bash
pnpm create react-app --template typescript sample-app
```

pnpmを無視してnpmで作成されます．

# peer dependencies

`node_modules`と`package-lock.json`を削除して再インストールします．

```bash
cd sample-app
pnpm i
```

すると，最後に「これらをインストールした方がいいよ」といわれるのでインストールします．

```bash
pnpm i @babel/plugin-syntax-flow @babel/plugin-transform-react-jsx @testing-library/dom @babel/core @babel/plugin-transform-react-jsx
```

# testing-libraryのエラー

このままでは`pnpm start`を実行してもエラーを吐きます．

```bash
Property 'toBeInTheDocument' does not exist on type 'JestMatchers<HTMLElement>'.
```

`@types/testing-library__jest-dom`を入れれば解決するらしい．

https://github.com/facebook/create-react-app/issues/12622

```bash
pnpm i @types/testing-library__jest-dom
```

# devDependencies

TypeScriptがdependenciesにいるのは個人的に気持ち悪いのでdevDependenciesに移動します．

```bash
pnpm i -D @babel/core @babel/plugin-syntax-flow @babel/plugin-transform-react-jsx @testing-library/dom @testing-library/jest-dom @testing-library/react @testing-library/user-event @types/jest @types/node @types/react @types/react-dom @types/testing-library__jest-dom typescript
```

# 絶対パスでのインポート

いちいちディレクトリを戻らなくていいように`src`ディレクトリから絶対パスでインポートできるようにします．

```diff json
  {
    "compilerOptions": {
      ...
+     "baseUrl": "./src"
    }
  }
```

Next.jsでは`@/components/Button`のような形でプレフィックスをつけることができますが，Reactではできないらしいです．

```tsx
import Button from 'components/Button'

function App() {
  return (
    ...
    <Button>Hello World!</Button>
  )
}
```

# ESLintの導入

ESLintのインストールと初期化を行います．

```bash
pnpm create @eslint/config
```

質問に答えながら設定していきます．

```
√ How would you like to use ESLint? · style
√ What type of modules does your project use? · esm
√ Which framework does your project use? · react
√ Does your project use TypeScript? · Yes
√ Where does your code run? · browser
√ How would you like to define a style for your project? · guide
√ Which style guide do you want to follow? · standard-with-typescript
√ What format do you want your config file to be in? · JSON
√ Would you like to install them now? · Yes
√ Which package manager do you want to use? · pnpm
```

追加のdependenciesをインストールします．

```bash
pnpm i -D @typescript-eslint/parser
```

すると`eslintrc.json`が作成されるので少しいじります．

```diff json:.eslintrc.json
  {
    "env": {
      "browser": true,
      "es2021": true
    },
    "extends": [
+     "react-app",
+     "react-app/jest",
      "plugin:react/recommended",
      "standard-with-typescript"
    ],
    "overrides": [],
+   "parser": "@typescript-eslint/parser",
    "parserOptions": {
      "ecmaVersion": "latest",
      "sourceType": "module",
+     "project": ["./tsconfig.json"]
    },
-   "plugins": ["react"],
+   "plugins": ["react", "@typescript-eslint", "import"],
    "rules": {
+     "space-before-function-paren": "off",
+     "@typescript-eslint/space-before-function-paren": "off",
+     "@typescript-eslint/no-floating-promises": "off",
+     "@typescript-eslint/explicit-function-return-type": "off"
    }
  }
```

# Prettierの導入

インストールと設定ファイルの作成をします．

```bash
pnpm i -D prettier eslint-config-prettier eslint-plugin-prettier
```

```json:.prettier.json
{
  "arrowParens": "always",
  "bracketSameLine": false,
  "bracketSpacing": true,
  "htmlWhitespaceSensitivity": "css",
  "insertPragma": false,
  "jsxSingleQuote": true,
  "printWidth": 100,
  "quoteProps": "as-needed",
  "rangeStart": 0,
  "requirePragma": false,
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "useTabs": false
}
```

`.eslintrc.json`を変更します．

```diff json:.eslintrc.json
  {
    ...
    "extends": [
      "react-app",
      "react-app/jest",
      "plugin:react/recommended",
      "standard-with-typescript",
+     "plugin:prettier/recommended"
    ],
    ...
  }
```

# Storybookの追加

Storybookはpnpmに対応していませんが，無理やり追加します．

```bash
pnpx sb init -s
```

途中で追加するものを聞かれますが，無視して後で手動で追加します．
