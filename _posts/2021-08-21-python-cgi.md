---
layout: post
title:  "Python でお手軽ローカル CGI"
date:   2021-08-21 15:30:00 +0900
categories: python
---

## 概要

Python で CGI を動かして軽くテストする環境を作りたい，というときの手順は，いくつかの記事に書いてある．

- [python でお気楽Webサーバを構築してCGIのテスト。 \- Qiita](https://qiita.com/goodboy_max/items/833d482827bf0efab45a)

しかし，上手く行ったり行かなかったりしたので，メモを書いておく．


## 環境

- Ubuntu 20.04.2 LTS (Focal Fossa)
  - `$ cat /etc/os-release` で確認
- Python 3.8.10
  - `$ python -V` で確認


## 結論

次のような構成でディレクトリ郡を作る．

```
hoge/
├ main.sh
└ cgi-bin/
    ├ fuga.py
    ├ piyo.py
    ├ ...
```

main.sh の中身はこんな感じ．

```sh
python -m http.server 8765 --cgi
```

fuga.py の中身はこんな感じ．

```py
#!/usr/bin/env python
# -*- coding:utf-8 -*-

print('Content-type: text/plain; charset=UTF-8\r\n')
print('hogehoge')
```

`$ bash ./main.sh` を実行して，Web ブラウザから `http://0.0.0.0:8765/cgi-bin/fuga.py` へアクセスすると，hogehoge と表示されるはず．


## トラブルシューティング

### OSError: [Errno 8] Exec format error

Web ブラウザから `http://0.0.0.0:8765/cgi-bin/fuga.py` へアクセスした瞬間，次のようなログがコンソールに出ることがある．

なお，この際，Web ブラウザへは `200 Script output follows` が返されているが，これは正常な動作である．

```
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 34994)
Traceback (most recent call last):
  File "/home/<username>/miniconda3/envs/<envname>/lib/python3.8/http/server.py", line 1179, in run_cgi
    os.execve(scriptfile, args, env)
OSError: [Errno 8] Exec format error: '/path/to/cgi-bin/fuga.py'
----------------------------------------
```

上記エラーの原因は，Python スクリプトに shebang を記載していないことである．すなわち，ファイル冒頭に

```py
#!/usr/bin/env python
# -*- coding:utf-8 -*-
```

を記せばよい．

- [Python 組み込みのCGIHTTPServerやhttp\.serverでcgiを動かす \- Symfoware](https://symfoware.blog.fc2.com/blog-entry-1977.html)


### レスポンスが空になっている

Web ブラウザに `200 Script output follows` が返ってきているものの，レスポンスが空になっていることがある．

原因は，HTTP ヘッダを出力していないことである．すなわち，body を print するより前に，たとえば

```py
print('Content-type: text/plain; charset=UTF-8\r\n')
```

を記せばよい．


### その他

ページによっては，「cgi で実行しようとするスクリプトに実行権限が必要」との記載があるが，記事作成時の検証では不要だった．

---

以上
