---
layout: post
title:  "Ubuntu カスタム URI スキームとデスクトップエントリ"
date:   2021-07-31 14:30:00 +0900
categories: ubuntu
---
## 概要

Ubuntu デスクトップ環境でカスタム URI スキームを用いる方法を記したメモ．

「カスタム URI スキーム」は，文脈次第では，次のような名称で呼ばれたりもするようである．

- カスタム URL スキーム
- 外部プロトコルリクエスト
- プロトコルハンドラ
- カスタム プロトコル（本当はプロトコルじゃない気がするけど）
- カスタム ブラウザ プロトコル
- Pluggable Protocol Handler
- Asynchronous Pluggable Protocols


## 環境

- Ubuntu 20.04.2 LTS (Focal Fossa)
    - `$ cat /etc/os-release` で確認


## 前章：.desktop ファイルとデスクトップエントリ

### 動機づけ

Ubuntu のデスクトップ環境を使っているとき，GPU のドライバなどに起因して，suspend が上手く動作しない場合がある．

たとえば，以下のようにして suspend しようとしても，画面が一瞬暗くなるだけで，すぐに元の画面に復帰してしまう．

- デスクトップ右上から「サスペンド」を選択
- `$ systemctl suspend`

そんなとき，以下のやり方なら上手く動作する．

```sh
$ sudo apt install pm-utils
$ sudo pm-suspend
```

`pm-suspend` するたびに `sudo` でパスワードを入れるのは大変なので，`pm-suspend` だけはパスワードなしでも実行できるようにするとよい．具体的には，`$ sudo visudo` して，`%sudo   ALL=(ALL:ALL) ALL` の次の行に `myusername    ALL=NOPASSWD: /usr/sbin/pm-suspend` を追加する．ここで，myusername は自身のユーザ名に置き換える．これで多少は楽になる．

- [sudo のパスワードを入力なしで使うには \- Qiita](https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7)

しかし，suspend する度にターミナルを開いて，コマンドを手打ちして，という手順を踏まねばならないのも嫌である．そこで，「アクティブティ」から "suspend" で検索できるようにして，そこからワンアクションで suspend できるようにしたい．


### デスクトップエントリの作成

初期状態では，`~/.local/share/applications` は空になっている．

まず，`~/.local/share/applications/mysuspend.desktop` を新規作成し，次のように中身を作成する：

```sh
[Desktop Entry]
Version=1.0
Type=Application
Name=My Suspender
Comment=Test MySuspend
Exec=bash -lc "sudo pm-suspend"
MimeType=x-scheme-handler/myprotocol
Terminal=true
```

bash は `-c` で入力を文字列から読み込む．`-l` で環境変数を読み込む．

これで，「アクティビティ」から "suspend" で検索すると「My Suspender」が現れるようになる．

なお，`desktop-file-install` を使う方法もあるようだが，ここでは紹介しない．


### ところで

ここまでで作った .desktop には，特定の MIME type を関連付けることができる．これを応用すれば，たとえばブラウザで "myprotocol:XXXXX" のような URI を開こうとしたときに，xdg-open 経由で自前の .desktop に処理を飛ばすことができる．


## カスタム URI スキーム

カスタムスキーム `myprotocol` を作成する．`~/.local/share/applications` は，初めは空になっている．

まず，`~/.local/share/applications/myprotocol.desktop` を新規作成し，次のように中身を作成する：

```sh
[Desktop Entry]
Version=1.0
Type=Application
Name=My Launcher
Comment=Test MyProtocol
Exec=bash -lc "python3 ~/git/workspaces/scripts/myprotocol.py %u"
MimeType=x-scheme-handler/myprotocol
Terminal=true
```

bash は `-c` で入力を文字列から読み込む．`-l` で環境変数を読み込む．

myprotocol.py の中身は，たとえば以下のような感じになる．

```python
import sys

uri: str = sys.argv[1]
print(uri)  # => "myprotocol:XXXXX"
input()
```

次に，以下を実行する．

```sh
$ cd ~/.local/share/applications
$ xdg-mime default myprotocol.desktop x-scheme-handler/myprotocol
```

これにより，`~/.config/mimeapps.list` に `x-scheme-handler/myprotocol=myprotocol.desktop` の記述が追加される．

```sh
# 実行権限を付与（←いらないっぽい）
# $ chmod +x myprotocol.desktop
$ gio mime x-scheme-handler/myprotocol myprotocol.desktop
# desktop ファイルでハンドルされる MIME types のキャッシュデータベースを構築する
$ sudo update-desktop-database
```

```sh
# 別のターミナルが開いたら成功
$ xdg-open 'myprotocol://abcd'
```

ターミナルから `xdg-open 'myprotocol://abcd'` すると環境変数など諸々が有効になるが，ブラウザやアクティブティから起動されると環境変数は効かないので，たとえば python とかを動かそうとしても，そのままでは動かない．

参考

- [gnome \- URL protocol handlers in basic Ubuntu Desktop \- Ask Ubuntu](https://askubuntu.com/questions/514125/url-protocol-handlers-in-basic-ubuntu-desktop/1023143#1023143)
- [command line \- Unable to create custom URI scheme on Ubuntu \- Ask Ubuntu](https://askubuntu.com/questions/961745/unable-to-create-custom-uri-scheme-on-ubuntu)


### デバッグ

たとえば，上記の例では `python3` にパスが通っていない場合，`xdg-open` で開かれたターミナルは瞬時に消えてしまう．そこで，下記のように sleep か何かを入れることで，ターミナルが消えないようにする．

```sh
Exec=bash -lc "python3 ~/git/workspaces/scripts/myprotocol.py %u ; sleep 100000;"
```


-----

以上
