---
layout: post
title:  "react-bootstrap-table2 メモ"
date:   2021-04-24 17:30:00 +0900
categories: react
---
## 概要

[AtCoder Marathon Replay](https://iilj.github.io/AtCoderMarathonReplay/#/standings/) を作るときに [react\-bootstrap\-table2 · Next Generation of react\-bootstrap\-table](https://react-bootstrap-table.github.io/react-bootstrap-table2/) を使ったので，2 じゃない react-bootstrap-table を使っていた身から，注意点などメモっておく．


## 移行

だいたい以下のページを読めば OK．`TableHeaderColumn` がなくなって，代わりに `columns` に列の情報を指定するようになったのが最大の変化．

- [Migration · react\-bootstrap\-table2](https://react-bootstrap-table.github.io/react-bootstrap-table2/docs/migration.html)


## ページネーション

- [Pagination Props · react\-bootstrap\-table2](https://react-bootstrap-table.github.io/react-bootstrap-table2/docs/pagination-props.html)

react-bootstrap-table に比べて，カスタマイズが容易になった．

たとえば，ページリストのボタンを変えたいなら，次のようにできる．この例では，リストに含まれるページ番号は既に決まっている．

- [Custom Page List - Storybook](https://react-bootstrap-table.github.io/react-bootstrap-table2/storybook/index.html?selectedKind=Pagination&selectedStory=Custom%20Page%20List&full=0&addons=1&stories=1&panelRight=0&addonPanel=storybook%2Factions%2Factions-panel)

また，ページリストの位置を変えたいときは，`PaginationListStandalone` を使う．

- [Cannot use pageListRenderer with PaginationListStandalone · Issue \#943 · react\-bootstrap\-table/react\-bootstrap\-table2](https://github.com/react-bootstrap-table/react-bootstrap-table2/issues/943)

では，たとえば AtCoder の順位表のように，全ページに対数回の操作で移動できるように，表示するページ番号を自分で決めたいようなときはどうするかというと，`PaginationProvider` というのを使う．

- [Fully Custom Pagination - Storybook](https://react-bootstrap-table.github.io/react-bootstrap-table2/storybook/index.html?selectedKind=Pagination&selectedStory=Fully%20Custom%20Pagination&full=0&addons=1&stories=1&panelRight=0&addonPanel=storybook%2Factions%2Factions-panel)


## 列プロパティ

以下ですべて紹介されている．

- [Columns Props · react\-bootstrap\-table2](https://react-bootstrap-table.github.io/react-bootstrap-table2/docs/column-props.html#columnformatter-function)


### ダミー列

同じデータフィールドの列を2つ以上用意すると warning が出る．

- [Two columns have same dataField will cause the same key warning of React · Issue \#601 · react\-bootstrap\-table/react\-bootstrap\-table2](https://github.com/react-bootstrap-table/react-bootstrap-table2/issues/601)

そんなときは `isDummyField: true` を使う．便利．

- [Action 1 and Action 2 are dummy column - Storybook](https://react-bootstrap-table.github.io/react-bootstrap-table2/storybook/index.html?selectedKind=Work%20on%20Columns&selectedStory=Dummy%20Column&full=0&addons=1&stories=1&panelRight=0&addonPanel=storybook%2Factions%2Factions-panel)

-----

以上
