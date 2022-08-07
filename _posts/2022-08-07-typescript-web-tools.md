---
layout: post
title:  "TypeScript で簡易 Web ツール開発"
date:   2022-08-07 18:00:00 +0900
categories: typescript
---

## 概要

Web ツール類を TypeScript で開発するための方法を記す．

今回は，こんな感じで，1つのサイト内に複数の Web ツールが入っているような形を目指す．

```
/ (root)
├ index.html
├ favicon.ico
├ tool1/
│　├ index.html
│　└ dist.js
├ tool2/
│　├ index.html
│　└ dist.js
├ tool3/
│　├ index.html
│　└ dist.js
︙
```


## 環境の準備

まずは以下の形を目指す．

```
hoge/
├ node_modules/
├ types/
├ dist/
├ src/
│　├ tool1/
│　│　├ main.ts
│　│　├ rollup.config.js
│　│　├ fuga.ts
│　│　︙
│　├ tool2/
│　│　├ main.ts
│　│　├ rollup.config.js
│　│　├ piyo.ts
│　│　︙
│　︙
├ static/
│　├ favicon.ico
│　├ index.html
│　├ tool1/
│　│　├ index.html
│　│　︙
│　├ tool2/
│　│　├ index.html
│　│　︙
│　︙
├ package.json
├ tsconfig.json
└ eslintrc.json
```

何はともあれ `init`.

```sh
$ npm init
package name: (hoge) 
version: (1.0.0) 
description: 説明文を入力する
entry point: (index.js) 
test command: jest
git repository: 
keywords: 
author: xxx
license: (ISC) MIT

$ npm install --save-dev typescript sass rollup prettier rollup-plugin-html rollup-plugin-scss@3 rollup-plugin-typescript ts-jest jest eslint-plugin-prettier eslint-config-prettier eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin @types/jest cpx http-server
```

package.json を次のようにする．具体的には，

- entry point (`"main": "index.js",`) を消す
- typeRoots を追加する
- scripts 内に bundle などを追加する
  - Linux のみサポートなら下記で OK.
  - Windows 上で開発したいなら，`"lint": "tsc --noEmit && eslint \"src/**/*.ts\"",` とかにする
    - `;` で区切ったままだと `Unknown compiler option '--noEmit;'. Did you mean 'noEmit'?` と言われる
    - eslint で探すファイルのパスをシングルクォートで囲んだままだと，`No files matching the pattern` を言われる
- repository, bugs, homepage に git の情報を追加する（あれば）
- jest の設定を追加する
  - package.json 以外の場所で設定したい人はそれでも OK
<!-- - devDependencies を追加する -->

```json
{
  "name": "hoge",
  "version": "1.0.0",
  "description": "説明文を入力する",
  "typeRoots": [
    "types",
    "node_modules/@types"
  ],
  "scripts": {
    "copy": "cpx \"static/**/*.{html,ico}\" dist",
    "prettier": "prettier --write $PWD/static/'**/*.{js,jsx,ts,tsx,vue,css,html}'",
    "lint": "tsc --noEmit; eslint --ignore-path .gitignore \"**/*.{ts,tsx}\"",
    "lint:fix": "tsc --noEmit; eslint --ignore-path .gitignore \"**/*.{ts,tsx}\" --fix",
    "bundle": "rollup -c $npm_config_fn",
    "build": "npm run lint:fix && npm run prettier && npm run copy && rollup -c $npm_config_fn",
    "serve": "http-server -o /dist/",
    "test": "jest"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/xxx/hoge.git"
  },
  "author": "iilj",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/xxx/hoge/issues"
  },
  "homepage": "https://github.com/xxx/hoge#readme",
  "devDependencies": {
    "@types/jest": "^27.0.2",
    "@typescript-eslint/eslint-plugin": "^5.1.0",
    "@typescript-eslint/parser": "^5.1.0",
    "cpx": "^1.5.0",
    "eslint": "^8.0.1",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^4.0.0",
    "http-server": "^14.0.0",
    "jest": "^27.3.1",
    "prettier": "^2.4.1",
    "rollup": "^2.58.0",
    "rollup-plugin-html": "^0.2.1",
    "rollup-plugin-scss": "^3.0.0",
    "rollup-plugin-typescript": "^1.0.1",
    "sass": "^1.43.2",
    "ts-jest": "^27.0.7",
    "typescript": "^4.4.4"
  },
  "jest": {
    "moduleFileExtensions": [
      "ts",
      "js"
    ],
    "transform": {
      "^.+\\.ts$": "ts-jest"
    },
    "globals": {
      "ts-jest": {
        "tsconfig": "tsconfig.json"
      }
    },
    "testMatch": [
      "<rootDir>/src/**/*.test.ts"
    ]
  }
}
```

rollup.config.js を作成する．ここでは，src/main.ts をエントリポイントにしている．なお，以下の点に留意すること．

- `@match` の記述は適宜変更すること．
- `@exclude`, `@require`, `@resource` などは適宜追加すること．
- `@grant` の記述は適宜変更すること．

```js
import typescript from "rollup-plugin-typescript";
import html from "rollup-plugin-html";
import scss from 'rollup-plugin-scss';

export default [
    {
        input: "src/hoge/main.ts",
        output: {
            file: "dist/hoge/dist.js"
        },
        plugins: [
            html({
                include: "**/*.html"
            }),
            scss({
                output: false
            }),
            typescript()
        ]
    }
];
```

tsconfig.json を作成する．`es2017` では新しすぎる場合は，`es6` あたりにしておく．`strict` は様子を見ながら，大丈夫そうなら `true` にする．

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es2017",
        "typeRoots": [
            "./types",
            "./node_modules/@types"
        ],
        "strict": true
    },
    "exclude": [
        "node_modules"
    ]
}
```

.eslintrc.json を作成する．なお eslint の "indent" は prettier のインデントとバッティングするので，指定しないで prettier に任せる．

- [javascript \- Prettier and eslint indents not working together \- Stack Overflow](https://stackoverflow.com/questions/56337176/prettier-and-eslint-indents-not-working-together)

```json
{
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended",
        "plugin:@typescript-eslint/recommended-requiring-type-checking",
        "plugin:prettier/recommended"
    ],
    "plugins": [
        "@typescript-eslint"
    ],
    "env": {
        "browser": true,
        "es2017": true
    },
    "parser": "@typescript-eslint/parser",
    "parserOptions": {
        "sourceType": "module",
        "project": "./tsconfig.json",
        "ecmaVersion": 2017
    },
    "rules": {
        // "indent": [
        //     "error",
        //     4
        // ],
        "@typescript-eslint/no-use-before-define": [
            "error",
            {
                "functions": false,
                "classes": false
            }
        ],
        "prettier/prettier": [
            "error",
            {
                "printWidth": 120,
                "tabWidth": 4,
                "singleQuote": true
            }
        ]
    }
}
```


## 開発

rollup.config.js に記載した .ts ファイル（今回は src/main.ts) をエントリポイントとしてコードを記述していく．

main.ts の中身は，たとえば次のような感じで OK．

```ts
import { Hoge } from './hogehoge';

(() => {
    // 何かする
    Hoge.doSomething();
    console.log('Hello World!');
})();
```

適宜，types ディレクトリに *.d.ts を追加していく．


### CSS, HTML の import

たとえば，rollup-plugin-scss 経由で scss の中身を string として import したいとき，types/scss.d.ts を作成し，次のように記述する．

```ts
declare module "*.scss" {
    const content: string;
    export default content;
}
```

import する側は，通常通り次のような書き方で OK. これで，rollup されるときには，scss の記法から css の記法に変換されたものが変数 css の中身に入るので，

```ts
import css from './hoge.scss';

export const hoge = (): void => {
    GM_addStyle(css);
};
```

細かい設定を変えたいときは，rollup.config.js の中身を弄る．

HTML の中身を string として読みたいときも，大まかな手順は同じで，types/html.d.ts など適当な名前で定義ファイルを作成すればよい．


## Linter

```sh
$ npm rum lint:fix
```


## バンドル

以下を実行すると，dist/dist.js に内容が吐かれる．

```sh
$ npm run bundle
```


## ブラウザに読ませる

開発時は，次のようなものを用意して，通常時に使うほうを無効化すると便利．

- [ac\-predictor/ac\-predictor\-extension at master · key\-moon/ac\-predictor](https://github.com/key-moon/ac-predictor/tree/master/ac-predictor-extension)

```js
// ==UserScript==
// @name        hoge-dev
// @namespace   iilj
// @version     1.0.0
// @description 開発用のスクリプトです。ローカルからビルド済みスクリプトをロードします。
// @author      iilj
// @license     MIT
// @require     file://[リポジトリの親ディレクトリへのパス]/hgoe/dist/bundle.js
// @supportURL  https://github.com/xxx/hoge/issues
// @run-at      document-end
// @match       https://example.com/*
// ==/UserScript==
```

ただし，拡張機能に対してローカルファイルへのアクセスを許可する必要がある．ブラウザの拡張機能一覧画面から，拡張機能ごとに設定を行える．

- [Tampermonkey • FAQ](https://www.tampermonkey.net/faq.php#Q204)

なお，上記スクリプトは，rollup.config.js に記載したのと同じように，`@exclude`, `@require`, `@resource`, `@grant` などを設定する必要がある．

また，ファイルの場所は `file:///home/hoge/...` のように，通常はルートから記述する．

以上
