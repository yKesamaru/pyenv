# 旧システムvenvで作成された仮想環境中のプロジェクトをpyenvで動作させる手順

## はじめに
少し前にシステムを`Ubuntu 20.04`から`Ubuntu 22.04`へ移行しました。
新システムに移行にあたり、`Python`のバージョンが`3.8`から`3.10`へ変化しました。

このとき旧システムの`python仮想環境(venv)`で作成されたプロフェクトフォルダをアクティベートしようとすると、新システムのPythonが起動されてしまいます。
`ROOT/bin/python3`がシステムPythonへのシンボリックリンクになっているからです。（`ROOT`はプロジェクトのルートパス）
旧システムの`venv仮想環境のpythonバージョン`が合っていないと、この仮想環境を使用できません。

ではどうするかというと、`pyenv`で旧システムのPythonをエミュレートするか、`GNOME Boxes`で`Ubuntu 20.04`環境を作成し、その中でプロフェクトフォルダを動作させるか、になります。

そしてここでは、`pyenv`を使って旧システムのプロジェクトを動作させることを目指します。

しかし単純に`pyenv`で`python仮想環境(venv)`のPythonバージョンに合わせたPythonをインストールするだけでは旧システムの`venv環境`が動作してくれません。
**結論から先に書くと、`ROOT/bin/python3`, `ROOT/bin/python`のシンボリックリンクを`pyenv`由来のシンボリックリンクに置き換える必要があります。**

この記事では備忘録を兼ね、ステップバイステップでこの手順を記録します。

![](https://raw.githubusercontent.com/yKesamaru/pyenv/master/assets/eye-catch.png)

- [旧システムvenvで作成された仮想環境中のプロジェクトをpyenvで動作させる手順](#旧システムvenvで作成された仮想環境中のプロジェクトをpyenvで動作させる手順)
  - [はじめに](#はじめに)
  - [環境](#環境)
  - [`pyenv`インストール](#pyenvインストール)
    - [依存関係のインストール](#依存関係のインストール)
    - [`pyenv`本体のインストール](#pyenv本体のインストール)
  - [`.profile`, `.bash_profile`, `.bashrc`へ追加記述](#profile-bash_profile-bashrcへ追加記述)
    - [`.profile`](#profile)
    - [`.bash_profile`](#bash_profile)
    - [`.bashrc`](#bashrc)
    - [補足](#補足)
    - [`pyenv`動作確認](#pyenv動作確認)
  - [`任意バージョンのPython`導入](#任意バージョンのpython導入)
    - [インストール可能なバージョンリストの取得](#インストール可能なバージョンリストの取得)
    - [インストールするべきバージョンの確認](#インストールするべきバージョンの確認)
    - [バージョンを指定して`pyenv Python`をインストール](#バージョンを指定してpyenv-pythonをインストール)
      - [インストール確認](#インストール確認)
  - [仮想環境アクティベート](#仮想環境アクティベート)
    - [仮想環境中の`python3`リンクを変更](#仮想環境中のpython3リンクを変更)
  - [参考文献](#参考文献)


## 環境
```bash
$ inxi -Sxxx --filter
System:
  Kernel: 6.5.0-41-generic x86_64 bits: 64 compiler: N/A Desktop: GNOME 42.9
    tk: GTK 3.24.33 wm: gnome-shell dm: GDM3 42.0
    Distro: Ubuntu 22.04.4 LTS (Jammy Jellyfish)
```

## `pyenv`インストール

`pyenv`公式
https://github.com/pyenv/pyenv

### 依存関係のインストール
ビルドに必要なパッケージは事前にインストールしておきます。

Suggested build environment
https://github.com/pyenv/pyenv/wiki#suggested-build-environment

```bash
sudo apt update; sudo apt install build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```

### `pyenv`本体のインストール

Automatic installer
https://github.com/pyenv/pyenv?tab=readme-ov-file#automatic-installer

```bash
curl https://pyenv.run | bash
```

## `.profile`, `.bash_profile`, `.bashrc`へ追加記述


上記それぞれのファイルに以下を記述します。

Set up your shell environment for Pyenv
https://github.com/pyenv/pyenv?tab=readme-ov-file#set-up-your-shell-environment-for-pyenv

記述後は`source`コマンドで強制読み込みをするか、再ログインします。
わたしの場合、`source`コマンドで不具合が出ました。再ログインのほうが（理由はわかりませんが）確実かもしれません。

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


### 補足
これらのファイルは再インストール時、バックアップファイルから自動的にコピーされるようにしています。
詳しくは以下をご参照ください。

- システム再インストールを楽にする手順: 専用BashScriptのすすめ
  https://zenn.dev/ykesamaru/articles/1ab1297354d3c2

### `pyenv`動作確認
```bash
$ pyenv --version
pyenv 2.4.7
```

## `任意バージョンのPython`導入
Usage
https://github.com/pyenv/pyenv?tab=readme-ov-file#usage


### インストール可能なバージョンリストの取得
```bash
$ pyenv install --list > python_list.txt
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

### インストールするべきバージョンの確認
`venv`ですでに仮想環境が作成されている状態なのですが、`pyenv`でこれに適合したバージョンのPythonをインストールするときは、**マイクロバージョンまで一致したバージョン**である必要があります。

例えば`3.8系`の最終バージョンは`3.8.19`ですが、旧システムの`venv`で作成した仮想環境は`3.8.10`です。
もし`pyenv`で`3.8.19`をインストールしても`venv仮想環境`と適合しないためプロジェクトを動作させることはできません。
旧システムの`venv仮想環境`のマイクロバージョンを調べるには、以下のファイルをロードします。
```bash
$ cat ~/bin/FACE01/pyenv.cfg
home = /usr/bin
include-system-site-packages = false
version = 3.8.10
```
これで`3.8.10`であることが確認できます。

### バージョンを指定して`pyenv Python`をインストール
マイクロバージョンが`3.8.10`と判明したのでこれをインストールします。
```bash
$ pyenv install 3.8.10
# コケたのでコンパイラフラグを指定してビルド
$ CFLAGS="-O2" pyenv install 3.8.10
# それでもコケたのでビルドフラグも追加
$ CFLAGS="-O2" CPPFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" pyenv install 3.8.10
Downloading Python-3.8.10.tar.xz...
-> https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tar.xz
Installing Python-3.8.10...
patching file Misc/NEWS.d/next/Build/2021-10-11-16-27-38.bpo-45405.iSfdW5.rst
patching file configure
patching file configure.ac
Installed Python-3.8.10 to /home/user/.pyenv/versions/3.8.10
```
上記の例ではコンパイラフラグとビルドフラグを設定してようやくビルドできました。
これは完全に環境に依存します。この作業は何度も行ったことがありますが、通常`pyenv install *version*`で失敗したことはありませんでした。
コケたときのログ（`/tmp/ディレクトリ以下）をみて判断してください。

#### インストール確認
`python 3.8.10`をローカルに指定
`$ pyenv local 3.8.10`
```bash
# バージョンを確認
$ python --version
Python 3.8.10
```
`3.8.10`がインストールされました。

## 仮想環境アクティベート
仮想環境はすでに`venv`で作成済みです。
ですので仮想環境のルートディレクトリに移動して、アクティベートします。
このとき、`pyenv`を`local`で指定して起動します。
```bash
$ cd ~/bin/FACE01
$ pyenv local 3.8.10
$ source bin/activate
```
先述の通り、このようにするとシステムPython(ver.3.10系)が立ち上がってしまいます。
これを回避するため、以下に続く作業を行います。

### 仮想環境中の`python3`リンクを変更
`ROOT/bin/`ディレクトリには、システムPythonへのリンクが存在します。
![](https://raw.githubusercontent.com/yKesamaru/pyenv/master/assets/2024-07-12-11-47-21.png)
`. bin/activate`を行うと、このリンクをたどってシステムPythonが起動してしまうため、これらのシンボリックリンクを変更します。
```bash
user@user:~/bin/FACE01/bin$ cd /home/user/bin/FACE01/bin  # プロジェクトのルートパス
# 一応バックアップをとっておく
user@user:~/bin/FACE01/bin$ mkdir python3.10系
user@user:~/bin/FACE01/bin$ mv python3 python3.10系/
user@user:~/bin/FACE01/bin$ mv python python3.10系/
# pyenvからシンボリックリンクを貼る
user@user:~/bin/FACE01/bin$ ln -s /home/user/.pyenv/versions/3.8.10/bin/python3 python3
user@user:~/bin/FACE01/bin$ ln -s /home/user/.pyenv/versions/3.8.10/bin/python python
```
仮想環境をアクティベートしてPythonのバージョンを確認します。
```bash
user@user:~/bin/FACE01$ . bin/activate
(FACE01) user@user:~/bin/FACE01$ python
Python 3.8.10 (default, Jul 12 2024, 08:29:07) 
[GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
これで旧システムの`venv`で作成された仮想環境を再び使用することができます。

## 参考文献
- [pyenv](https://github.com/pyenv/pyenv)
  - [Suggested build environment](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)
  - [Automatic installer](https://github.com/pyenv/pyenv?tab=readme-ov-file#automatic-installer)
  - [Set up your shell environment for Pyenv](https://github.com/pyenv/pyenv?tab=readme-ov-file#set-up-your-shell-environment-for-pyenv)
  - [Usage](https://github.com/pyenv/pyenv?tab=readme-ov-file#usage)
- [システム再インストールを楽にする手順: 専用BashScriptのすすめ](https://zenn.dev/ykesamaru/articles/1ab1297354d3c2)
