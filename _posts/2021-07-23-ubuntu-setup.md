---
layout: post
title:  "Ubuntu 新規環境セットアップのメモ"
date:   2021-07-23 14:30:00 +0900
categories: ubuntu
---
## 概要

Windows が入っているマシンに Ubuntu 環境を新たにセットアップするときの個人的手順書．


## 環境

- AMD Ryzen 7 3700X 8-Core Processor
    - `$ sudo lshw -class processor` で確認
- NVIDIA GeForce RTX 3060 12GB GDD R6
- Ubuntu 20.04.2 LTS (Focal Fossa)
    - `$ cat /etc/os-release` で確認


## インストール用メディア作成

1. USB メモリを用意する．
1. 以下のページから ubuntu-ja-20.04.1-desktop-amd64.iso を落とす．
    - [Ubuntu Desktop 日本語 Remixのダウンロード \| Ubuntu Japanese Team](https://www.ubuntulinux.jp/products/JA-Localized/download)
1. USB メモリを挿入し，`$ lsblk -f` でデバイス名およびマウントポイントを確認する．
    - 以下，デバイス名は `sdb` とする．
1. `$ umount "マウントポイント"` で対象 USB メモリをアンマウントする．
    - `umount` である．`unmount` ではない．
1. `$ dd bs=4M if=/path/to/ubuntu-ja-20.04.1-desktop-amd64.iso of=/dev/sdb status=progress && sync` で iso の中身を書き込む．
    - `of=/dev/sdb` のようにする．`of=/dev/sdb1` のようにすると起動できない．
        - [command line \- How to create a bootable Ubuntu USB flash drive from terminal? \- Ask Ubuntu](https://askubuntu.com/questions/372607/how-to-create-a-bootable-ubuntu-usb-flash-drive-from-terminal)
        - つまり，アンマウントするのは `sdb1` で，書き込み先は `sdb` とする．

参考

- [Linux上でisoファイルからUbuntuのLive USBメモリを作成する方法 \| 普段使いのArch Linux](https://www.archlinux.site/2018/03/linuxisoubuntulive-usb.html)
- [Ubuntu 20\.04 LTS インストールUSBを作る \| 二代目俺のメモ](https://www.kwonline.org/memo2/2020/04/24/create-ubuntu-20_04-lts-install-usb/)



## Boot 関係設定

pc 起動時のスプラッシュ画面で特定のキーを押すと，何かしらの画面に遷移できる．この設定はマシン依存．

- F11: Boot Select Menu
    - [Boot Select Menu を起動する方法 \| ドスパラ サポートFAQ よくあるご質問｜お客様の｢困った｣や｢知りたい｣にお応えします。](http://faq3.dospara.co.jp/faq/show/6289?site_domain=default)
- Delete: UEFI BIOS Utility
    - [ドスパラ サポートFAQ よくあるご質問｜お客様の｢困った｣や｢知りたい｣にお応えします。](http://faq3.dospara.co.jp/faq/wizard_select/9935?site_domain=default&wizard_id=5290)
    - たとえば，虹色に光るゲーミング PC を鎮めたり．


## Ubuntu インストール

- Live USB に初めから GParted が入っているので，Windows が入っているパーティションを切り詰めて，次の 2 つのパーティションを用意する．
    - `/` をマウントするドライブ
    - `/home` をマウントするドライブ
- インストール用ウィザードで「インストールの種類」が出てきたら，「それ以外」を選択する．
- パーティション一覧が出るので，先程作った2つのパーティションそれぞれ右クリック→「変更」から，マウントポイントを選択する．
    - 何もせずに次に進もうとすると，「ルートファイルシステムが定義されていません。パーティショニングメニューでこれを修正してください」と言われる．
        - [パーティションあれこれ: Linux Diary](http://maru89.cocolog-nifty.com/blog/2007/10/post_dd68.html)
        - [Ubuntu日本語フォーラム / ルートファイルシステムが定義されていません。](https://forums.ubuntulinux.jp/viewtopic.php?id=333)
    - スワップ領域は，今回は用意しなかった．
        - [2016年12月21日　スワップパーティションはもういらない ―Ubuntu，Zapusでは"Swapfiles"に：Linux Daily Topics｜gihyo\.jp … 技術評論社](https://gihyo.jp/admin/clip/01/linux_dt/201612/21)


## 各種セットアップ

- Visual Studio Code を .deb からインストールする
    - [Visual Studio Code \- Code Editing\. Redefined](https://code.visualstudio.com/)
- Vivaldi を .deb からインストールする
    - [Vivaldi をダウンロード \| Vivaldi Browser](https://vivaldi.com/ja/download/)
- apt 経由でいろいろ入れる
    - Dropbox は，nautilus-dropbox インストール後に一度立ち上げて daemon をインストールする必要がある．
    - gnome-shell-extension-system-monitor インストール後，再起動して拡張機能マネージャを開いて「system-monitor」を ON にする．
        - [システムモニターアプレットを Ubuntu 20\.04 タスクバーに表示する \- Qiita](https://qiita.com/nanbuwks/items/3bccce7f608c9c9091db)
        - このとき，トグルボタンが切り替えられない場合，ウィンドウ右上に全体切替のトグルボタンがあるので，それを切り替える．
            - [Question: I can't activate any of my extensions : gnome](https://www.reddit.com/r/gnome/comments/6gf5d4/question_i_cant_activate_any_of_my_extensions/)
- 他にも
    - Myrica は事前にダウンロードしておく
        - [プログラミングフォント Myrica / Estable \| Myrica （ミリカ）は、フリーなプログラミング用 TrueType フォントです。](https://myrica.estable.jp/)
        - [Ubuntuで最初にやる設定 \- ネガティブログ](https://xvideos.hatenablog.com/entry/ubuntu_setup)


```sh
$ sudo apt update

# git のインストール
$ sudo apt install git
$ git config --global user.email "you@example.com"
$ git config --global user.name "Your Name"

# Dropbox のインストール
# (インストール後に作業が必要)
$ sudo apt install nautilus-dropbox

# KeePassXC のインストール
$ sudo apt install keepassxc

# Ricty Dinimised のインストール
# $ sudo apt install fonts-ricty-diminished
# バッククォートがバグっていて面倒なので，Myrica にしておく

# Myrica のインストール
# 事前に Myrica.TTC をダウンロードしておく
$ sudo mkdir /usr/share/fonts/truetype/myrica
$ sudo mv Myrica.TTC /usr/share/fonts/truetype/myrica/
$ fc-cache -fv
$ fc-list | grep myrica

# ホームディレクトリのフォルダ名を日本語から英語に変更する
$ LANG=C xdg-user-dirs-gtk-update

# gnome-shell-extension-system-monitor のインストール
# (インストール後に作業が必要)
$ sudo apt install gnome-shell-extension-system-monitor

# CopyQ のインストール
$ sudo add-apt-repository ppa:hluk/copyq
$ sudo apt update
$ sudo apt install copyq

# g++-7 のインストール（C++17 対応）
$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
$ sudo apt update
$ sudo apt install g++-7
$ echo "export CXX='g++-7'" >> ~/.bashrc
$ echo "export CC='gcc-7'" >> ~/.bashrc

# boost のインストール
$ sudo apt install libboost-all-dev

# VLC のインストール
$ sudo apt install vlc vlc-plugin-access-extra libbluray-bdj libdvdcss2
$ sudo apt install ubuntu-restricted-extras
# $ sudo apt install libav-tools ffmpeg
# $ sudo apt install libdvdnav4 libdvdread7 gstreamer1.0-plugins-bad libdvd-pkg

# Gimp のインストール
$ sudo apt install gimp
```

参考

- [Ubuntu 18\.04にGitをインストールする方法](https://www.codeflow.site/ja/article/how-to-install-git-on-ubuntu-18-04)
- [ファイル同期の定番！「Dropbox」をUbuntuにインストールする方法！ \| LFI](https://linuxfan.info/dropbox-on-ubuntu)
- [Download \- KeePassXC](https://keepassxc.org/download/)
- [Ubuntuでc\+\+17のために最新版のg\+\+, boost, cmakeを自然に使う \- Qiita](https://qiita.com/forno/items/a877c3fd9d2056217042)
- [Ubuntu 20\.04 インストール \(5\) \- kashiの日記](http://verifiedby.me/adiary/0143)


### Vivaldi の同期が競合する

ログアウト→ログイン，で解決する．

- [Sync failing upload at the moment\. \| Vivaldi Forum](https://forum.vivaldi.net/topic/39784/sync-failing-upload-at-the-moment/8)


## CapsLock を Control に置換する

1. `$ sudo apt install gnome-tweaks`
1. Tweaks を起動し，「キーボードとマウス」→「追加のレイアウトオプション」→「Ctrl Position」→「Caps Lock を Ctrl として扱う」にチェックを入れる
1. ログアウトして再度ログインする

参考

- [Ubuntu/Caps\-LockキーをCtrlキーにする方法 \- Linuxと過ごす](https://linux.just4fun.biz/?Ubuntu/Caps-Lock%E3%82%AD%E3%83%BC%E3%82%92Ctrl%E3%82%AD%E3%83%BC%E3%81%AB%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95)
- なお，/etc/default/keyboard を編集する方法もあるようだが，こちらの環境では何も変化しなかった．
    - [Ubuntu 16\.04 CapsLockをControlに置換する方法 \- Qiita](https://qiita.com/hirooooooaki/items/f404e76c6f171769412a)


## トップバーの時刻表示に秒数を追加する

1. 前節でインストールした Tweaks を開く
1. 「トップバー」で「秒」のトグルスイッチを ON にする


## マウスホイールのスクロール速度を速くする

- [Ubuntuでマウスホイールのスクロール速度を上げる \- 人生シーケンスブレイク](https://shnsprk.com/entry/2020/04/09/090000)

`~/.imwheelrc` を作成し，内容を以下のようにする．

```
".*"
None,      Up,   Button4, 8
None,      Down, Button5, 8
```

余談だが，ファイル名を `~/.imwheel` と紹介しているページもある．しかし `~/.imwheelrc` でないと動かない．

```sh
$ sudo apt install imwheel
# 起動
$ imwheel
```

PC 起動時に自動で imwheel を起動するように設定する．

- [Ubuntuでimwheelでマウスホイールのスクロール速度の変更](https://codechacha.com/ja/linux-imwheel/)



## GPU 関係諸々インストール

### NVIDIA Driver インストール

サウンドの設定で出力デバイスが「ダミー出力」になっているとき，ドライバを入れないと音が出ない．機種によるが，Wi-Fi や Bluetooth も同じ．

- [ドスパラ製ゲーミングノートにUbuntuを入れる時の罠と対策 \- Qiita](https://qiita.com/cyubachi/items/a0237f7ae5aca1214c20)

事前にセキュアブートを切っておき，以下を実行する．

```sh
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt update

$ ubuntu-drivers devices
# 上の結果で出てくるドライバ一覧のうち，末尾に recommended と書いてあるドライバを入れる

$ sudo apt install nvidia-driver-470
# セキュアブートが入っている場合，インストールが進むと，Enroll MOK のときに使うパスワードを登録数画面が出てくるので，入力する．

$ sudo reboot
# セキュアブートが入っている場合，起動時に表示されるメニューで「Enroll MOK」を選択→「Continue」→「Yes」→パスワード入力→「Reboot」
```

- [UbuntuにNVIDIA driverをインストール/再インストールする方法 \- Qiita](https://qiita.com/yto1292/items/463e054943f3076f36cc)

ここで，NVIDIA driver のバージョンを確認しておく．`470.57.02` の部分．

```sh
$ cat /proc/driver/nvidia/version
NVRM version: NVIDIA UNIX x86_64 Kernel Module  470.57.02  Tue Jul 13 16:14:05 UTC 2021
GCC version:  gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04) 
```

なおバージョンについては，下記ページに必要事項を入力して「検索」を押下したときに「バージョン:」欄に表示されるものと同じになっていることを確認する．

- [NVIDIAドライバダウンロード](https://www.nvidia.co.jp/Download/index.aspx?lang=jp)


### CUDA インストール

新しすぎるバージョンの CUDA だと，PyTorch が対応していなかったりして困るので，今回は最新の 11.4 ではなく 11.1 を入れる．

```sh
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
$ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
$ sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
$ sudo apt update
# $ sudo apt install cuda
$ sudo apt install cuda-runtime-11-1 cuda-demo-suite-11-1 cuda-drivers cuda-drivers-470 cuda-11-1
$ sudo reboot
```

参考

- [CUDA Toolkit 11\.1 Update 1 Downloads \| NVIDIA Developer](https://developer.nvidia.com/cuda-11.1.1-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=2004&target_type=debnetwork)
- バージョン指定の話
    - [Ubuntu CUDAをaptでインストール \+ cuDNNのインストール \- Symfoware](https://symfoware.blog.fc2.com/blog-entry-2439.html)
    - [Ubuntu 20\.04へのCUDAインストール方法 \- Qiita](https://qiita.com/yukoba/items/c4a45435c6ee5d66706d)

以下を ~/.bashrc に追記すると，nvcc などが使えるようになる．

```sh
export PATH="/usr/local/cuda/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
```

再起動して CUDA バージョンを確認する．`11.1.105` の部分．

```sh
$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2020 NVIDIA Corporation
Built on Mon_Oct_12_20:09:46_PDT_2020
Cuda compilation tools, release 11.1, V11.1.105
Build cuda_11.1.TC455_06.29190527_0
```

次のページの Table 1 を見て，CUDA と Driver のバージョンが整合しているか確認する．

- [Release Notes :: CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)

基本的に，

- 新しい CUDA は新しい Driver を要求する
- 新しい Driver は古い CUDA でも動く

という対応になっている．nvidia-driver のバージョンを確認して，対応する CUDA を入れることになる．CUDA が新しすぎると駄目，という構図．


### 間違えて別のバージョンを入れてしまったとき

```sh
$ sudo apt purge cuda
$ sudo apt purge nvidia*
$ sudo apt purge libnvidia*
$ sudo apt purge ./cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
$ sudo apt autoremove
$ sudo apt autoclean
$ sudo apt update
```

- [【新規環境/既存環境対応】最新のCUDA及びcuDNNのインストール方法 \- Qiita](https://qiita.com/harmegiddo/items/86b295ccf96eff489e02)


### Finished with code: 256

症状としては以下と同じ．

- [\[INFO\]: Finished with code: 256 , \[ERROR\]: Install of driver component failed \- CUDA / CUDA Setup and Installation \- NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/info-finished-with-code-256-error-install-of-driver-component-failed/107661)

CUDA ダウンロードページの，「runfile (local)」を参照していると，こうなるかもしれない．そのときは代わりに，「deb (local)」あるいは「deb (network)」を参照してインストールする．

### Existing package manager installation of the driver found.

「runfile (local)」を参照してインストールしようとしていると，発生するかもしれない．「deb (local)」あるいは「deb (network)」を参照してインストールする．

- [Error installing Cuda toolkit: Existing package manager installation of the driver found \- Ask Ubuntu](https://askubuntu.com/questions/1211919/error-installing-cuda-toolkit-existing-package-manager-installation-of-the-driv)
- [Tensorflow2\.4 Install to Ubuntu20\.04 備忘録 \- Qiita](https://qiita.com/Ihmon/items/4ed0bd013435aac16bdc)


### CuDNN インストール

NVIDIA Developer のアカウントを作成し，ログインのうえ以下のページから必要なものをダウンロードできる．

- [cuDNN Download \| NVIDIA Developer](https://developer.nvidia.com/rdp/cudnn-download)
- [cuDNN Archive \| NVIDIA Developer](https://developer.nvidia.com/rdp/cudnn-archive)

今回は以下の3つ．

- cuDNN Runtime Library for Ubuntu20.04 x86_64 (Deb)
- cuDNN Developer Library for Ubuntu20.04 x86_64 (Deb)
- cuDNN Code Samples and User Guide for Ubuntu20.04 x86_64 (Deb)

```sh
$ sudo apt install ./libcudnn8_8.2.2.26-1+cuda11.4_amd64.deb
$ sudo apt install ./libcudnn8-dev_8.2.2.26-1+cuda11.4_amd64.deb
$ sudo apt install ./libcudnn8-samples_8.2.2.26-1+cuda11.4_amd64.deb
```

CUDA と CuDNN のバージョン対応については以下のサポート表を参照．

- [Support Matrix :: NVIDIA Deep Learning cuDNN Documentation](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html)
- [NVidiaドライバ・CUDA・cuDNNのバージョン対応 \+ tensorflowの対応 \- programming\-notes](https://scrapbox.io/programming-notes/NVidia%E3%83%89%E3%83%A9%E3%82%A4%E3%83%90%E3%83%BBCUDA%E3%83%BBcuDNN%E3%81%AE%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E5%AF%BE%E5%BF%9C_+_tensorflow%E3%81%AE%E5%AF%BE%E5%BF%9C)
- [ソースからのビルド  \|  TensorFlow](https://www.tensorflow.org/install/source?hl=ja#gpu)
- [NvidiaドライバとCUDAとcuDNNとTensorflow\-gpuとPythonのバージョンの対応 \- Qiita](https://qiita.com/konzo_/items/a6f2e8818e5e8fcdb896)

参考

- [UbuntuにcuDNNをインストール \- Qiita](https://qiita.com/tatsuya11bbs/items/70205b070c7afd7dd651)


### 間違えて別のバージョンを入れてしまったとき

```sh
$ sudo apt purge ./libcudnn8-samples_8.2.2.26-1+cuda11.4_amd64.deb
$ sudo apt purge ./libcudnn8-dev_8.2.2.26-1+cuda11.4_amd64.deb
$ sudo apt purge ./libcudnn8_8.2.2.26-1+cuda11.4_amd64.deb
$ sudo apt autoremove
$ sudo apt autoclean
$ sudo apt update
```


## Miniconda 環境構築

1. 以下のページから Miniconda3-py38_4.10.3-Linux-x86_64.sh をダウンロードする．
    - [Miniconda — Conda documentation](https://docs.conda.io/en/latest/miniconda.html#linux-installers)
1. `$ bash Miniconda3-py38_4.10.3-Linux-x86_64.sh`
    - 基本的に yes でインストールを進めていけば OK
1. ターミナルを再起動，(base) が立ち上がる．
1. `$ conda create -n si1 --clone base` で base 環境が si1 環境に複製される．
1. 以下を実行する．

```sh
# $ conda config --add channels conda-forge
# $ conda config --set channel_priority strict
# $ conda install -c conda-forge numpy scipy scikit-learn matplotlib opencv
# $ conda install -c anaconda cupy
# $ conda install -c pytorch pytorch
$ pip install numpy scipy scikit-learn matplotlib opencv-python autopep8
$ pip install cupy-cuda111
$ pip install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 -f https://download.pytorch.org/whl/torch_stable.html
```

参考

- [Miniconda Install 備忘録 \- Qiita](https://qiita.com/Ihmon/items/11074e1a4c0e397d934f)
- cupy のバージョン指定について
    - [Installation — CuPy 9\.2\.0 documentation](https://docs.cupy.dev/en/stable/install.html)
- torch バージョン関係は公式に案内されているものに従う
    - [PyTorch](https://pytorch.org/)
    - [Start Locally \| PyTorch](https://pytorch.org/get-started/locally/)


### fatal error: cublas_v2.h

CUDA やら CuDNN やらを入れないで cupy を入れようとすると，多分こうなる．先にバージョンを考えて CUDA を入れる．

- [No such file or directory \#include <cublas\_v2\.h> \- C\+\+ \- PyTorch Forums](https://discuss.pytorch.org/t/no-such-file-or-directory-include-cublas-v2-h/73204)


## 競プロ環境セットアップ

```sh
# oj 関係
$ pip install online-judge-tools
$ pip install selenium
$ sudo apt install chromium-chromedriver firefox-geckodriver
$ oj login https://atcoder.jp/
$ oj login https://yukicoder.me/

# nodejs 関係
# atcoder-cli を動かすのに使う＋開発に使う
$ sudo apt install nodejs npm
$ sudo npm install n -g
$ sudo n stable
$ sudo apt purge -y nodejs npm
$ exec $SHELL -l
$ node -v

# atcoder-cli 関係
$ sudo npm install -g atcoder-cli
$ acc -h
$ acc login
$ acc session

# Rust 関係
# AHC 公式ビジュアライザ・ジェネレータは大抵は Rust で書かれている
$ sudo apt install curl
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Ruby 関係
$ sudo apt install ruby
# どうせ使うので入れる
$ sudo gem install nokogiri
```

参考

- atcoder-cli 関係
    - [oj/getting\-started\.ja\.md at master · online\-judge\-tools/oj](https://github.com/online-judge-tools/oj/blob/master/docs/getting-started.ja.md)
    - [atcoder\-cli チュートリアル \| わたしろぐ](http://tatamo.81.la/blog/2018/12/07/atcoder-cli-tutorial/)
- Rust 関係
    - [Install Rust \- Rust Programming Language](https://www.rust-lang.org/tools/install)
    - [Ubuntu 20\.04 LTS に Rustを新規インストール \- Qiita](https://qiita.com/ayumi-ikeda/items/c3b4d6997079dcd2f4a2)


### Userscript 導入

- [ac\-predictor](https://greasyfork.org/ja/scripts/369954-ac-predictor)
- [ac\-writers script](https://greasyfork.org/ja/scripts/369965-ac-writers-script)
- [AtCoder Difficulty Display](https://greasyfork.org/ja/scripts/397185-atcoder-difficulty-display)
- [AtCoder Rating Cumulative Distribution](https://greasyfork.org/ja/scripts/419055-atcoder-rating-cumulative-distribution)
- [AtCoder TestCase Extension](https://greasyfork.org/ja/scripts/371832-atcoder-testcase-extension)
- [atcoder\-standings\-difficulty\-analyzer](https://greasyfork.org/ja/scripts/419541-atcoder-standings-difficulty-analyzer)
- [atcoder\-standings\-lang](https://greasyfork.org/ja/scripts/415894-atcoder-standings-lang)
- [atcoder\-tasks\-page\-colorize\-during\-contests](https://greasyfork.org/ja/scripts/426049-atcoder-tasks-page-colorize-during-contests)
- [atcoder\-tasks\-page\-colorizer](https://greasyfork.org/ja/scripts/380404-atcoder-tasks-page-colorizer)
- [atcoder\-wait\-time\-display](https://greasyfork.org/ja/scripts/430509-atcoder-wait-time-display)
- [AtcoderDevotionGraph](https://greasyfork.org/ja/scripts/416588-atcoderdevotiongraph)
- [AtCoderPerformanceColorizer](https://greasyfork.org/ja/scripts/371693-atcoderperformancecolorizer)
- [AtCoderResultNotifier](https://greasyfork.org/ja/scripts/371225-atcoderresultnotifier)
- [AtCoderStandingsAnalysis](https://greasyfork.org/ja/scripts/398439-atcoderstandingsanalysis)
- [comfortable\-yukicoder](https://greasyfork.org/ja/scripts/431129-comfortable-yukicoder)
- [RationalModConverterUserJS](https://greasyfork.org/ja/scripts/425982-rationalmodconverteruserjs)
- [Time Limit Emphasizer](https://greasyfork.org/ja/scripts/406381-time-limit-emphasizer)
<!-- - ★ HOJ User Emphasizer -->


### vscode 各種設定

- ターミナルへ移動するためのショートカットを登録する
    - "File"→"Preference"→"Keyboard Shortcuts" で "terminal.focus" で検索
    - "workbench.action.terminal.focus" にショートカットを設定する
    - [Visual Studio Code でエディタとターミナルを移動するショートカットキーの作成 \- Qiita](https://qiita.com/Shun141/items/fbed88abd4518f6d4039)
- 画面に合わせて表示倍率を調整する
    - [【VSCode】ウインドウ全体を拡大／縮小／リセットする](http://maniera.xyz/2019/11/29/post-425/)
- 各種拡張機能インストール
    - C/C++
    - Python
    - Ruby
    - Code Runner
        - [Code Runnerを使いこなす \- Qiita](https://qiita.com/take_me/items/6a1d2d417889837219d1#%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%AE%E5%A4%89%E6%9B%B4)
    - Colonize
    - #region folding for VS Code

```
"cpp": "cd $dir && g++ $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",

"cpp": "cd $dir && g++ - $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
```

### Formatting failed. See the output window for details. Source: C/C++ (Extension)

「書式設定が失敗しました。詳細については、出力ウィンドウを参照してください。」とポップアップが表示されるとき，OUTPUT ウィンドウのドロップダウンボックスを「Tasks」から「C/C++」に切り替えると，エラーの内容がわかる．

- [Formatting Failed C/C\+\+ No info in Output window · Issue \#3271 · microsoft/vscode\-cpptools](https://github.com/microsoft/vscode-cpptools/issues/3271)


## suspend できないとき

NVIDIA のドライバの関係で，suspend が上手く行かないことがある．

- [UbuntuにGeForceを入れてサスペンドできない場合の解決例 \- Qiita](https://qiita.com/ocean_f/items/60825042f10c7f43a4af)
- [Ubuntu 18\.04 LTSとNVIDIAドライバーの組み合わせでサスペンド・レジュームができるようにする \| Terakinizers' Tells](https://www.terakin.com/ja/blog/archives/134)

まず以下で再起動できるようになるか試す．

```sh
$ sudo apt install pm-utils
$ sudo pm-suspend
```

駄目ならディスプレイマネージャを変えてみる．

```
$ sudo apt install lightdm
```


## Usercss

- [ツイッター：プロモーション（広告）を非表示にする \| Userstyles\.org](https://userstyles.org/styles/118856/theme)


-----

以上
