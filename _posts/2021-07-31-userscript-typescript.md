---
layout: post
title:  "Userscript を TypeScript で書く"
date:   2021-07-31 18:00:00 +0900
categories: userscript
---

## 概要

UserScript を TypeScript で開発するための方法を記す．

方法はいくつかあるが，GrasyFork などに投稿する際には minify や難読化がかかっていてはいけないので，その点に引っかからないように bundle する必要がある．


## 環境の準備

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

$ npm install --save-dev typescript sass rollup prettier rollup-plugin-html rollup-plugin-scss@3 rollup-plugin-typescript ts-jest jest eslint-plugin-prettier eslint-config-prettier eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin @types/jest @types/tampermonkey
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
    "lint": "tsc --noEmit; eslint 'src/**/*.ts'",
    "lint:fix": "tsc --noEmit; eslint --fix 'src/**/*.ts'",
    "bundle": "rollup -c",
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
    "@types/jest": "^26.0.24",
    "@types/tampermonkey": "^4.0.2",
    "@typescript-eslint/eslint-plugin": "^4.28.5",
    "@typescript-eslint/parser": "^4.28.5",
    "eslint": "^7.32.0",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^3.4.0",
    "jest": "^27.0.6",
    "prettier": "^2.3.2",
    "rollup": "^2.55.1",
    "rollup-plugin-html": "^0.2.1",
    "rollup-plugin-scss": "^3.0.0",
    "rollup-plugin-typescript": "^1.0.1",
    "sass": "^1.37.5",
    "ts-jest": "^27.0.4",
    "typescript": "^4.3.5"
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
import packageJson from "./package.json";

const userScriptBanner = `
// ==UserScript==
// @name         ${packageJson.name}
// @namespace    xxx
// @version      ${packageJson.version}
// @description  ${packageJson.description}
// @author       ${packageJson.author}
// @license      ${packageJson.license}
// @supportURL   ${packageJson.bugs.url}
// @match        https://example.com/*
// @grant        none
// ==/UserScript==`.trim();

export default [
    {
        input: "src/main.ts",
        output: {
            banner: userScriptBanner,
            file: "dist/dist.js"
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

なお，`@types/tampermonkey` をインストールしない場合は，types ディレクトリを作成し，以下の tampermonkey-module.d.ts を複製する．

- [userscript\-typescript\-webpack/tampermonkey\-module\.d\.ts at master · vannhi/userscript\-typescript\-webpack](https://github.com/vannhi/userscript-typescript-webpack/blob/master/tampermonkey-module.d.ts)


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
