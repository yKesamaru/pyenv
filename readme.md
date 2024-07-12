# pyenvセットアップ

## はじめに
使い慣れた`pyenv`ですが、新システム(`Ubuntu 22.04`)へ移行してからセットアップをしていませんでした。
これは再インストール用のスクリプトの該当部分をコメントアウトしていたためです。
新システムに移行にあたり、`Python`のバージョンが`3.8`から`3.10`へ変化しました。
次回のシステム移行はまだ随分と先になりそうですが、そういう場合に限って「前どうやったっけ？」となるのは火をみるより明らかです。

ここでは再インストール時の`pyenv`の導入を見据えて、今回の導入作業の記録をとりたいと思います。

![](assets/eye-catch.png)


## 環境
```bash
$ inxi -Sxxx --filter
System:
  Kernel: 6.5.0-41-generic x86_64 bits: 64 compiler: N/A Desktop: GNOME 42.9
    tk: GTK 3.24.33 wm: gnome-shell dm: GDM3 42.0
    Distro: Ubuntu 22.04.4 LTS (Jammy Jellyfish)
```

## インストール
Automatic installer
https://github.com/pyenv/pyenv?tab=readme-ov-file#automatic-installer

```bash
curl https://pyenv.run | bash
```

## `.profile`, `.bash_profile`, `.bashrc`
これらのファイルは再インストール時、バックアップファイルから自動的にコピーされます。
詳しくは以下をご参照ください。

- システム再インストールを楽にする手順: 専用BashScriptのすすめ
  https://zenn.dev/ykesamaru/articles/1ab1297354d3c2

- `pyenv`公式
  https://github.com/pyenv/pyenv

上記それぞれのファイルに以下を記述します。

Set up your shell environment for Pyenv
https://github.com/pyenv/pyenv?tab=readme-ov-file#set-up-your-shell-environment-for-pyenv

### `.profile`
```bash
# pyenv初期化
export PYENV_ROOT="$HOME/.pyenv"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```
### `.bash_profile`
```bash
# pyenv初期化
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```
### `.bashrc`
```bash
# pyenvの設定
# pyenvの設定
export PYENV_ROOT="$HOME/.pyenv"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```
記述後は`source`コマンドで強制読み込みをするか、再ログインします。
わたしの場合、`source`コマンドで不具合が出ました。再ログインのほうが（理由はわかりませんが）確実かもしれません。

### 動作確認
```bash
$ pyenv --version
pyenv 2.4.7
```

## `pyenv`導入
Usage
https://github.com/pyenv/pyenv?tab=readme-ov-file#usage

### 依存関係のインストール
ビルドに必要なパッケージは事前にインストールしておきます。

Suggested build environment
https://github.com/pyenv/pyenv/wiki#suggested-build-environment

```bash
sudo apt update; sudo apt install build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```

### インストール可能なバージョンリストの取得
```bash
pyenv install --list > python_list.txt
```
```bash
Available versions:
  2.1.3
  2.2.3
  2.3.7
(中略)
  stackless-3.4-dev
  stackless-3.4.2
  stackless-3.4.7
  stackless-3.5.4
  stackless-3.7.5
```
824種類の形式・バージョンがありました。
`Ubuntu 20.04`での`Python`のバージョンは`3.8`系列なので、最終安定版の`3.8.19`をインストールします。
```bash
$ pyenv install 3.8.19
Downloading Python-3.8.19.tar.xz...
-> https://www.python.org/ftp/python/3.8.19/Python-3.8.19.tar.xz
Installing Python-3.8.19...

BUILD FAILED (Ubuntu 22.04 using python-build 2.4.7)

Inspect or clean up the working tree at /tmp/python-build.20240711212627.34085
Results logged to /tmp/python-build.20240711212627.34085.log

Last 10 log lines:
0x735a17a29d8f __libc_start_call_main
        ../sysdeps/nptl/libc_start_call_main.h:58
0x735a17a29e3f __libc_start_main_impl
        ../csu/libc-start.c:392
Please submit a full bug report,
with preprocessed source if appropriate.
Please include the complete backtrace with any bug report.
See <file:///usr/share/doc/gcc-11/README.Bugs> for instructions.
make: *** [Makefile:1730: Objects/dictobject.o] エラー 1
make: *** 未完了のジョブを待っています....
```
インストールに失敗しました。
ログを確認したところ、`-03`フラグが指定されていたので、`-02`フラグを指定して再インストールしてみます。
```bash
$ CFLAGS="-O2" pyenv install 3.8.19
Downloading Python-3.8.19.tar.xz...
-> https://www.python.org/ftp/python/3.8.19/Python-3.8.19.tar.xz
Installing Python-3.8.19...
Installed Python-3.8.19 to /home/terms/.pyenv/versions/3.8.19
```
無事インストールできました。

## 仮想環境アクティベート
仮想環境はすでに`venv`で作成済みです。
ですので仮想環境のルートディレクトリに移動して、アクティベートします。
このとき、`pyenv`を`local`で指定して起動します。
```bash
$ cd ~/bin/FACE01
$ pyenv local 3.8.19
$ source bin/activate
```
仮想環境アクティベート自体はこの方法で適切なのですが、旧システムで構築した仮想環境のマイクロバージョンが一致していなくて、仮想環境中のライブラリを使用することができませんでした。

マイナーバージョンまで一致していれば良いと考えていたのですが、間違っていたようです。

マイクロバージョンを調べるには以下のようにします。
マイクロバージョンを調べる際、仮想環境をアクティベートしたまま`which`コマンドを使います。
```bash
$  . bin/activate
(FACE01) terms@terms:~/bin/FACE01$ which python
/home/terms/bin/FACE01/bin/python
```
アクティベートしていない状態だと以下のようになります。
```bash
$ which python
~/.pyenv/shims/python
```

さて、マイクロバージョンを調べるには、以下のファイルをロードします。
```bash
$ cat ~/bin/FACE01/pyenv.cfg
home = /usr/bin
include-system-site-packages = false
version = 3.8.10
```

マイクロバージョンが`3.8.10`と判明したのでこれをインストールします。
```bash
# コンパイラフラグを指定してビルド
$ CFLAGS="-O2" pyenv install 3.8.10
# コケたのでビルドフラグも追加
$ CFLAGS="-O2" CPPFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" pyenv install 3.8.10
Downloading Python-3.8.10.tar.xz...
-> https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tar.xz
Installing Python-3.8.10...
patching file Misc/NEWS.d/next/Build/2021-10-11-16-27-38.bpo-45405.iSfdW5.rst
patching file configure
patching file configure.ac
Installed Python-3.8.10 to /home/terms/.pyenv/versions/3.8.10
```

`pyenv`によって使用される`python`の現在のバージョンを一応確認。
```bash
$ cat .python-version
3.8.19
```
これを削除
`$ rm .python-version`
`python 3.8.10`をローカルに指定
`$ pyenv local 3.8.10`
```bash
# バージョンを確認
$ python --version
Python 3.8.10
```

## 仮想環境中の`python3`リンクを変更
`ROOT/bin/`ディレクトリには、システムPythonへのリンクが存在します。
`. bin/activate`を行うと、このリンクをたどってシステムPythonが起動してしまうため、これらのシンボリックリンクを変更します。
```bash
terms@terms:~/bin/FACE01/bin$ cd /home/terms/bin/FACE01/bin  # プロジェクトのルートパス
# 一応バックアップをとっておく
terms@terms:~/bin/FACE01/bin$ mkdir python3.10系
terms@terms:~/bin/FACE01/bin$ mv python3 python3.10系/
terms@terms:~/bin/FACE01/bin$ mv python python3.10系/
# pyenvからシンボリックリンクを貼る
terms@terms:~/bin/FACE01/bin$ ln -s /home/terms/.pyenv/versions/3.8.10/bin/python3 python3
terms@terms:~/bin/FACE01/bin$ ln -s /home/terms/.pyenv/versions/3.8.10/bin/python python
```
仮想環境をアクティベートしてPythonのバージョンを確認します。
```bash
terms@terms:~/bin/FACE01$ . bin/activate
(FACE01) terms@terms:~/bin/FACE01$ python
Python 3.8.10 (default, Jul 12 2024, 08:29:07) 
[GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
