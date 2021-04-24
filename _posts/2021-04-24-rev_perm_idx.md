---
layout: post
title:  "辞書順で N 番目の順列を求める"
date:   2021-04-24 17:30:00 +0900
categories: competitive_programming
---
## 概要

次の問題を解くことを考える．

- [No\.1311 Reverse Permutation Index \- yukicoder](https://yukicoder.me/problems/no/1311)

この問題は，次の3段階で解くことができる．

1. 長さ $S$ の順列のうち，辞書順で $N$ 番目のものを求める
    - たとえば，長さ $5$ の順列のうち，辞書順で $50$ 番目のものは $(3, 1, 4, 2, 5)$ である
2. 1\. で求めた順列の逆置換（=単調増加列にするための置換）を求める
    - たとえば，順列 $(3, 1, 4, 2, 5)$ に対して置換 $(2, 4, 1, 3, 5)$ を適用すると，順列 $(1, 2, 3, 4, 5)$ になる
    - これは容易に求められる
3. 2\. で求めた置換を順列とみなし，その順列が辞書順で何番目か求める
    - たとえば，順列 $(2, 4, 1, 3, 5)$ は，長さ $5$ の順列のうち，辞書順で $36$ 番目である
    - 既出：[K \- 辞書順で何番目？](https://atcoder.jp/contests/chokudai_S001/tasks/chokudai_S001_k)

ここで難関になるのは，「1. 長さ $S$ の順列のうち，辞書順で $N$ 番目のものを求める」の部分である．


## 辞書順で N 番目の順列を求める

コンテスト本番中は，下記ページを参考にしてプログラムを書いた．

- [順列を一瞬で取得するプログラム \- 目標は商店街をつくる事なんです。](http://itakanaya9.hatenablog.com/entry/2014/02/17/121428)

曰く，次のような手順で，長さ $S$ の順列のうち，辞書順で $N$ 番目の順列を得ることができる．

1. $N$ を $1$ で割り，その商をさらに $2$ で割り，その商をさらに $3$ で割り，…と繰り返し，商を $S$ で割るまで繰り返す．このとき，各除算で出た余りを $r_1, r_2, \dots, r_S$ とする．
2. 数列 $A = (1, 2, \dots, S)$ および空数列 $B$ を用意し，$k$ を $r_S$ から $r_1$ まで順に辿りながら，次の操作をする．
    - 数列 $A$ から， 0-indexed で $k$ 番目の要素を取り除き，$B$ の末尾に追加する．
3. 2\. の操作が終了したときの数列 $B$ が，長さ $S$ の順列のうち，辞書順で $N$ 番目のものになっている．

もう少しちゃんと書くと，こうである．

1. for $d=1$ to $S$:
  - $r_i \leftarrow N \bmod d$
  - $N \leftarrow N\\:\mathrm{div}\\:d$
2. $A = (1, 2, \dots, S)$
3. $B = ()$
4. for $i=S$ to $1$:
  - $k \leftarrow r_i$
  - $B$.append($A_k$)
  - $A$.remove($A_k$)
5. return $B$

なんのこっちゃという話である．


## 順列を求める例

睨んでもわからないので，例を追っていこう．

長さ $5$ の順列のうち，辞書順で $50$ 番目のものを求めてみよう．

除算を繰り返すところは，こうなる．

- $50 \div \textcolor{blue}{1} = 50 \cdots \textcolor{red}{0}$ → $r_1$
- $50 \div \textcolor{blue}{2} = 25 \cdots \textcolor{red}{0}$ → $r_2$
- $25 \div \textcolor{blue}{3} = 8 \cdots \textcolor{red}{1}$ → $r_3$
- $8 \div \textcolor{blue}{4} = 2 \cdots \textcolor{red}{0}$ → $r_4$
- $2 \div \textcolor{blue}{5} = 0 \cdots \textcolor{red}{2}$ → $r_5$

したがって，$(r_5, r_4, r_3, r_2, r_1) = (2, 0, 1, 0, 0)$ である．

ここで，数列 $(1, 2, 3, 4, 5)$ を用意して，この中から要素を順番に選び出していく．

- $(1, 2, 3, 4, 5)$ のうち，0-indexed で $2$ 番目は $\textcolor{red}{3}$
- $(1, 2, 4, 5)$ のうち，0-indexed で $0$ 番目は $\textcolor{red}{1}$
- $(2, 4, 5)$ のうち，0-indexed で $1$ 番目は $\textcolor{red}{4}$
- $(2, 5)$ のうち，0-indexed で $0$ 番目は $\textcolor{red}{2}$
- $(5)$ のうち，0-indexed で $0$ 番目は $\textcolor{red}{5}$

よって求める順列は $(3, 1, 4, 2, 5)$ である．冒頭の例で載せた答えと一致する．


## なぜ上手くいくのか

上述の方法がなぜ上手くいくのか，考えてみる．上の $(3, 1, 4, 2, 5)$ の例をまた使ってみる．

長さ $5$ の順列は，全部で $5! = 120$ 通りである．これらは，次のように分類できる．

- $(1, ?,?,?,?)$ 型 … $4! = 24$ 通り
- $(2, ?,?,?,?)$ 型 … $4! = 24$ 通り
- $(3, ?,?,?,?)$ 型 … $4! = 24$ 通り
- $(4, ?,?,?,?)$ 型 … $4! = 24$ 通り
- $(5, ?,?,?,?)$ 型 … $4! = 24$ 通り

ここで，$(3, ?,?,?,?)$ 型についてさらに分類してみると次のようになる．

- $(3, 1, ?,?,?)$ 型 … $3! = 6$ 通り
- $(3, 2, ?,?,?)$ 型 … $3! = 6$ 通り
- $(3, 4, ?,?,?)$ 型 … $3! = 6$ 通り
- $(3, 5, ?,?,?)$ 型 … $3! = 6$ 通り

ここで，$(3, 1, ?,?,?)$ 型についてさらに分類してみると次のようになる．

- $(3, 1, 2, ?,?)$ 型 … $2! = 2$ 通り
- $(3, 1, 4, ?,?)$ 型 … $2! = 2$ 通り
- $(3, 1, 5, ?,?)$ 型 … $2! = 2$ 通り

ここで，$(3, 1, 4, ?,?)$ 型についてさらに分類してみると次のようになる．

- $(3, 1, 4, 2, ?)$ 型 … $1! = 1$ 通り
- $(3, 1, 4, 5, ?)$ 型 … $1! = 1$ 通り

ところで，除算を繰り返していた部分では，こんな計算をしていたのだった．

- $50 \div \textcolor{blue}{1} = 50 \cdots \textcolor{red}{0}$
- $50 \div \textcolor{blue}{2} = 25 \cdots \textcolor{red}{0}$
- $25 \div \textcolor{blue}{3} = 8 \cdots \textcolor{red}{1}$
- $8 \div \textcolor{blue}{4} = 2 \cdots \textcolor{red}{0}$
- $2 \div \textcolor{blue}{5} = 0 \cdots \textcolor{red}{2}$

上記の式は，以下の形で表現できる．

- $50 = \textcolor{blue}{1} \times 50 + \textcolor{red}{0}$
- $50 = \textcolor{blue}{2} \times 25 + \textcolor{red}{0}$
- $25 = \textcolor{blue}{3} \times 8 + \textcolor{red}{1}$
- $8 = \textcolor{blue}{4} \times 2 + \textcolor{red}{0}$
- $2 = \textcolor{blue}{5} \times 0 + \textcolor{red}{2}$

上の式たちは，ユークリッドの互除法のときによくあるやり方と同じように，下の式を上の式に埋め込むような形で，下の方から順に，次のよう整理できる．

- $8 = \textcolor{blue}{4} \times \left(\textcolor{blue}{5} \times 0 + \textcolor{red}{2}\right) + \textcolor{red}{0}$
- $25 = \textcolor{blue}{3} \times \left(\textcolor{blue}{4} \times \left(\textcolor{blue}{5} \times 0 + \textcolor{red}{2}\right) + \textcolor{red}{0}\right) + \textcolor{red}{1}$
- $50 = \textcolor{blue}{2} \times \left(\textcolor{blue}{3} \times \left(\textcolor{blue}{4} \times \left(\textcolor{blue}{5} \times 0 + \textcolor{red}{2}\right) + \textcolor{red}{0}\right) + \textcolor{red}{1}\right) + \textcolor{red}{0}$
- $50 = \textcolor{blue}{1} \times \left(\textcolor{blue}{2} \times \left(\textcolor{blue}{3} \times \left(\textcolor{blue}{4} \times \left(\textcolor{blue}{5} \times 0 + \textcolor{red}{2}\right) + \textcolor{red}{0}\right) + \textcolor{red}{1}\right) + \textcolor{red}{0}\right) + \textcolor{red}{0}$

整理して最後に出てきた式は，展開して整理できる．

- $50 = \textcolor{blue}{1} \times \left(\textcolor{blue}{2} \times \left(\textcolor{blue}{3} \times \left(\textcolor{blue}{4} \times \left(\textcolor{blue}{5} \times 0 + \textcolor{red}{2}\right) + \textcolor{red}{0}\right) + \textcolor{red}{1}\right) + \textcolor{red}{0}\right) + \textcolor{red}{0}$
- $50 = \textcolor{blue}{1} \times \left(\textcolor{blue}{2} \times \left(\textcolor{blue}{3} \times \left(\textcolor{blue}{4} \times \textcolor{blue}{5} \times 0 + \textcolor{blue}{4} \times \textcolor{red}{2} + \textcolor{red}{0}\right) + \textcolor{red}{1}\right) + \textcolor{red}{0}\right) + \textcolor{red}{0}$
- $50 = \textcolor{blue}{1} \times \left(\textcolor{blue}{2} \times \left(\textcolor{blue}{3} \times \textcolor{blue}{4} \times \textcolor{blue}{5} \times 0 + \textcolor{blue}{3} \times \textcolor{blue}{4} \times \textcolor{red}{2} + \textcolor{blue}{3} \times \textcolor{red}{0} + \textcolor{red}{1}\right) + \textcolor{red}{0}\right) + \textcolor{red}{0}$
- $50 = \textcolor{blue}{1} \times \left(\textcolor{blue}{2} \times \textcolor{blue}{3} \times \textcolor{blue}{4} \times \textcolor{blue}{5} \times 0 + \textcolor{blue}{2} \times \textcolor{blue}{3} \times \textcolor{blue}{4} \times \textcolor{red}{2} + \textcolor{blue}{2} \times \textcolor{blue}{3} \times \textcolor{red}{0} + \textcolor{blue}{2} \times \textcolor{red}{1} + \textcolor{red}{0}\right) + \textcolor{red}{0}$
- $50 = \textcolor{blue}{1} \times \textcolor{blue}{2} \times \textcolor{blue}{3} \times \textcolor{blue}{4} \times \textcolor{blue}{5} \times 0 + \textcolor{blue}{1} \times \textcolor{blue}{2} \times \textcolor{blue}{3} \times \textcolor{blue}{4} \times \textcolor{red}{2} + \textcolor{blue}{1} \times \textcolor{blue}{2} \times \textcolor{blue}{3} \times \textcolor{red}{0} + \textcolor{blue}{1} \times \textcolor{blue}{2} \times \textcolor{red}{1} + \textcolor{blue}{1} \times \textcolor{red}{0} + \textcolor{red}{0}$
- $50 = \textcolor{blue}{5!} \times 0 + \textcolor{blue}{4!} \times \textcolor{red}{2} + \textcolor{blue}{3!} \times \textcolor{red}{0} + \textcolor{blue}{2!} \times \textcolor{red}{1} + \textcolor{blue}{1!} \times \textcolor{red}{0} + \textcolor{red}{0}$

さて，意味ありげな式が出てきたので，この意味を考えていく．$50 = \textcolor{blue}{5!} \times 0 + \textcolor{blue}{4!} \times \textcolor{red}{2} + \textcolor{blue}{3!} \times \textcolor{red}{0} + \textcolor{blue}{2!} \times \textcolor{red}{1} + \textcolor{blue}{1!} \times \textcolor{red}{0} + \textcolor{red}{0}$ というのはつまり，

- $50$ という数字は，
  - $\textcolor{blue}{5!}$ が $0$ 個（あたりまえ）
  - $\textcolor{blue}{4!}$ が $\textcolor{red}{2}$ 個
  - $\textcolor{blue}{3!}$ が $\textcolor{red}{0}$ 個
  - $\textcolor{blue}{2!}$ が $\textcolor{red}{1}$ 個
  - $\textcolor{blue}{1!}$ が $\textcolor{red}{0}$ 個
  - 最後に $\textcolor{red}{0}$ が余る（ひとまず無視）
- と分解することができる

ということを意味している．これは，

- $\textcolor{blue}{5!}$ が $0$ 個
  - 長さ $5$ の順列は全部で $5!$ 通りしかないので，当然 $0$ になる
  - あえていうなら，$(?, ?,?,?,?)$ 型 であることを主張している
- $\textcolor{blue}{4!}$ が $\textcolor{red}{2}$ 個
  - $(1, ?,?,?,?)$ 型, $(2, ?,?,?,?)$ 型 は通り過ぎるので，$(3, ?,?,?,?)$ 型 であることがわかる
- $\textcolor{blue}{3!}$ が $\textcolor{red}{0}$ 個
  - $(3, ?,?,?,?)$ 型 の中で，$\textcolor{blue}{3!}$ を1つも作れないということは，$(3, 1, ?,?,?)$ 型 である
- $\textcolor{blue}{2!}$ が $\textcolor{red}{1}$ 個
  - $(3, 1, ?,?,?)$ 型 のうち，$(3, 1, 2, ?,?)$ 型 は通り過ぎるので，$(3, 1, 4, ?,?)$ 型 であることがわかる
- $\textcolor{blue}{1!}$ が $\textcolor{red}{0}$ 個
  - $(3, 1, 4, ?,?)$ 型 の中で，$\textcolor{blue}{1!}$ を1つも作れないということは，$(3, 1, 4, 2, ?)$ 型である
- 最後に $\textcolor{red}{0}$ が余る
  - $(3, 1, 4, 2, ?)$ 型 のうち，最初のもの，すなわち $(3, 1, 4, 2, 5)$ 型 が条件を満たす

という意味である．以上により，上述のアルゴリズムが正当化されるのである．

ここまで辿れば当然わかることではあるが，アルゴリズム前半の「除算を繰り返す部分」は，実は $S!$ で割る → $(S-1)!$ で割る → $(S-2)!$ で割る → … → $2!$ で割る → $1!$ で割る，という操作を効率的に行っているに過ぎない．

-----

以上
