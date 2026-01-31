---
title: "create next-app で勝手に導入されるeslint-config-nextが何をしてるのか調べてみた"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "eslint"]
published: true
---

## はじめに

create next-appすると勝手にESLintがインストールされてセットアップされるのはいいのですが、ESLintプラグインを追加して設定しようとした時にFlatConfigをどう変更すればいいのか、壊してしまいそうで臆してしまうことありませんか？そこで今回は勝手に設定される`eslint-config-next`の中身がどうなっているのかを、[next.js/packages/eslint-config-next/src/index.ts at canary · vercel/next.js](https://github.com/vercel/next.js/blob/canary/packages/eslint-config-next/src/index.ts)を読みつつChatGPTに質問しながら紐解いてみました。

## import

```ts
// plugins
import next from '@next/eslint-plugin-next'
import react from 'eslint-plugin-react'
import reactHooks from 'eslint-plugin-react-hooks'
import tsEslint from 'typescript-eslint'
// import * as ... for plugins without default export
import * as importPlugin from 'eslint-plugin-import'
import * as jsxA11yPlugin from 'eslint-plugin-jsx-a11y'


// utils
import globals from 'globals'
import eslintParser from './parser'
```

まずはライブラリのインポート部分。独自のNext.jsプラグイン(`@next/eslint-plugin-next`)に加えて、`eslint-plugin-react`、`eslint-plugin-react-hooks`、`typescript-eslint`、`eslint-plugin-import`、`eslint-plugin-jsx-a11y`、がインポートされてます。

## rules

```ts
    rules: {
      ...react.configs.recommended.rules,
      ...reactHooks.configs.recommended.rules,
      ...next.configs.recommended.rules,
      'import/no-anonymous-default-export': 'warn',
      'react/no-unknown-property': 'off',
      'react/react-in-jsx-scope': 'off',
      'react/prop-types': 'off',
      'jsx-a11y/alt-text': [
        'warn',
        {
          elements: ['img'],
          img: ['Image'],
        },
      ],
      'jsx-a11y/aria-props': 'warn',
      'jsx-a11y/aria-proptypes': 'warn',
      'jsx-a11y/aria-unsupported-elements': 'warn',
      'jsx-a11y/role-has-required-aria-props': 'warn',
      'jsx-a11y/role-supports-aria-props': 'warn',
      'react/jsx-no-target-blank': 'off',
    },
```

次にrulesを確認すると、`eslint-plugin-react`、`eslint-plugin-react-hooks`、`@next/eslint-plugin-next`についてはrecommendedルールを指定しています。それに加えて個別のルールが設定されています。

### import/no-anonymous-default-export: 'warn'

無名のdefault exportの禁止

### react/no-unknown-property: 'off'

DOMに存在しないprops名を許可（禁止を無効化）。TailwindcssやStyledComponentsなどのライブラリでは`tw`や`css`などのpropsを渡すことがあるからでしょうか。

### react/react-in-jsx-scope: 'off'

旧来のReactでは`import React from 'react'`という記述が必要だったのですが、現在では不要になったためoffにしているようです。

### react/prop-types: 'off'

`prop-types`によるランタイムでの型チェックをoffにしています。TypeScriptでの型チェックがあるのでこっちはオフにしているみたい。

### jsx-a11y/alt-text

`<img>`に`alt`属性がないと警告を出すルール。Next.jsの`<Image>`コンポーネントに対しても`<img>`と同様に扱う設定がされてます。

その他アクセシビリティ系のルールは割愛。

### react/jsx-no-target-blank: 'off'

`target="_blank"`をつけたリンクに`rel="noopener noreferrer"`をつけないくてもOKにしている。
よくわからないがなにやら`next/link`と組み合わせた`<a>`の扱いが複雑なためらしい。

## languageOptions

```ts
languageOptions: {
  parser: eslintParser,
  parserOptions: {
    requireConfigFile: false,
    sourceType: 'module',
    allowImportExportEverywhere: true,
    // TODO: Is this needed?
    babelOptions: {
      presets: ['next/babel'],
      caller: {
        // Eslint supports top level await when a parser for it is included. We enable the parser by default for Babel.
        supportsTopLevelAwait: true,
      },
    },
  },
  globals: {
    ...globals.browser,
    ...globals.node,
  },
},
```

`languageOptions`はESLintがコードを解釈する際の設定です。使用するパーサーの指定と細かい指示、並びにグローバル変数の指定や`ecmaVersion`などを指定する項目です。

### parser

eslintParserは`./parser.ts`からインポートされています。

```ts
import type { Linter } from 'eslint'
// @ts-expect-error - No types for compiled modules.
import { parse, parseForESLint } from 'next/dist/compiled/babel/eslint-parser'
import { version } from '../package.json'

const parser: Linter.Parser = {
  parse,
  parseForESLint,
  meta: {
    name: 'eslint-config-next/parser',
    version,
  },
}

// Use `export =` instead of `export default` for ESLint parser compatibility.
// ESLint expects parser modules to be directly importable as CommonJS modules (module.exports).
export = parser
```

Babelのパーサーをラップしたもののようです。

### parserOptions.babelOptions

- `presets: ['next/babel']`: Next.jsが公式に提供しているBabelプリセットを使用しています。これによって、Next.jsの構文をESLintが理解できるようになるようです。
- `caller.supportsTopLevelAwait: true`: ESLintからBabelを呼ぶ時に、トップレベルawaitのサポートを伝える。

`// TODO: Is this needed?`というコメントが付いてますが不要なんでしょうかね 👀

### globals

```ts
globals: {
  ...globals.browser,
  ...globals.node,
},
```

globalsパッケージからブラウザとNode.js両方のグローバル変数を登録している。これによって`window`や`process`などが未定義の変数扱いされなくなる。Next.jsではサーバーとクライアント両方のコードを扱うため、ブラウザとNode.js両方を設定しているらしい。

## settings

```ts
settings: {
  react: {
    version: 'detect',
  },
  'import/parsers': {
    '@typescript-eslint/parser': ['.ts', '.mts', '.cts', '.tsx', '.d.ts'],
  },
  'import/resolver': {
    node: {
      extensions: ['.js', '.jsx', '.ts', '.tsx'],
    },
    typescript: {
      alwaysTryTypes: true,
    },
  },
},
```

`settings`は各プラグインごとの設定です。

### react

`eslint-plugin-react`に対する設定。
`version: 'detect'`: インストールされているReactのバージョンを自動検出して、バージョンに応じたルールを適用してくれる

### import/parsers

`eslint-plugin-import`において、TypeScriptファイルはどのパーサーで読むか設定。
`@typescript-eslint/parser`を使用する。

### import/resolver

```ts
  'import/resolver': {
    node: {
      extensions: ['.js', '.jsx', '.ts', '.tsx'],
    },
    typescript: {
      alwaysTryTypes: true,
    },
  },
```

同じく`eslint-plugin-import`において、import先のパスをどう解決するか設定。
Node.js的に`./foo`→`foo.js` | `foo.jsx` | `foo.ts` | `foo.tsx` と探索し、次にTypeScript的に`@/components/Button`や`@types/xxx`といった解決を試します。tsconfig.jsonの`paths`や`baseUrl`が関連してきます。