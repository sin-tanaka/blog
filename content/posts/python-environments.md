---
title: "今さらPythonの環境構築について知ってることまとめ【社内向け】"
date: 2019-11-11T09:38:09+09:00
draft: true
---

2019-11-11時点で自分が知ってることについてまとめておこうと思います。

## はじめに

大前提として状況に応じて使い分けましょう。

データ解析分野では、「とりあえずconda」という風潮もあるかもしれません。Python2 / 3系ぐらい大きなくくりで問題ない人もいれば、少なくとも3.6以上という人や2.6を使わなければいけないという状況もあります。

熟練者は今更とやかくいいません。多分なんでも使いこなした上です。

声高に反論してはいけませんし、お前を信じる仮想環境を信じましょう。

# 最初にまとめ

さまざまな仮想環境ツールがありますが、基本的には、

- PATHに仮想環境作成時のpythonのsymlinkを追加する
- sys.pathにsite-packagesを追加する

を実現するためのツールなので必要に応じて使い分けましょう。

## venv

venvでの仮想環境構築を通しておおまかな仮想環境の動きを追ってみます。

[venv \-\-\- 仮想環境の作成 — Python 3\.8\.0 ドキュメント](https://docs.python.org/ja/3/library/venv.html)

※ 2.7等には入ってません

公式の提供するモジュールです。3.3からデフォルトで使えます。

```sh
$ python3 -V  # Python 3.6.8

# python -m venv [自由になんでも名前]
$ python -m venv env
```

とすると[env_name]なディレクトリをカレントに作成します。これが仮想環境の実体です。

中身を見てみます。

```sh
$ tree -L 4
.
└── env
    ├── bin
    │   ├── activate
    │   ├── activate.csh
    │   ├── activate.fish
    │   ├── easy_install
    │   ├── easy_install-3.6
    │   ├── pip
    │   ├── pip3
    │   ├── pip3.6
    │   ├── python -> python3
    │   └── python3 -> /Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/bin/python3
    ├── include
    ├── lib
    │   └── python3.6
    │       └── site-packages
    └── pyvenv.cfg

5 directories, 11 files
```

bin以下にpythonの実体のパスに向いたsymbolic link、起動a
やpipがあるのがわかると思います。

仮想環境を有効化してみます。

起動コマンドですが、OSはもちろん、shellごとにも異なったりするので(たとえば.fishとか.cshがあるのが見えると思います)、公式Doc参照のこと。

```sh
$ source ./env/bin/activate
```

さて何が変わったのでしょうか

```sh
env > $ python -V  # Python 3.6.8
```

さきほどまで `python3` としていたコマンドが `python` だけでよくなっています。

これは仮想環境有効化の前後のPATHをみることで理解できます


```sh 
env > $ echo $PATH
/Users/yourusername/foobar_proj/env/bin:/usr/local/Cellar/zplug/2.4.2/bin:・・・

env > $ deactivate
/usr/local/Cellar/zplug/2.4.2/bin:・・・
```

仮想環境を有効化することで[env_name]以下のbinがPATHに追加されていますね。これが「仮想環境を有効化する」と表現していることの実体です。

外部のパッケージについて見てみます。

```sh
$ pip install requests
$ python
>>> import requests
>>> exit()
$ source ./env/bin/activate
env > $ python
>>> import requests
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'requests'
```

基本的に、仮想環境を作成した元のpythonで入れたパッケージは参照することができません。

よって、仮想環境内のpipを通してパッケージをインストールすることで仮想環境内でパッケージをimportできるようになります。

※「基本的に」と書いたのは、元のsite-packagesを参照するオプションがあるからです。

```sh
$ source env/bin/activate
env > $ pip install requests
env > $ python 
>>> import requests
```

こちらも仮想環境の有効化前後でimport pathの参照先を見てみます

```sh
env > $ python 
>>> import sys
>>> sys.path
['', '/Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/lib/python36.zip', '/Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6', '/Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6/lib-dynload', '/Users/yourusername/workspace_python/foobar/env/lib/python3.6/site-packages']
>>> exit()
env > $ deactivate
$ python
>>> import sys
>>> sys.path
['', '/Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/lib/python36.zip', '/Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6', '/Users/yourusername/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6/lib-dynload']
```

こちらも仮想環境下の `site-packages` が追加されていることがわかります。

以上の通り、venvはとてもシンプルな動きですし、プロジェクトのディレクトリ内に仮想環境を持てるというのは意味的にスッキリしています。

例えば、後述するpyenv-virtualenv等は単一の名前空間に仮想環境を作っていくので、プロジェクトとの関連は名前ぐらいです。

また、venvで作成した仮想環境ディレクトリはgit管理に含めないようにしましょう。絶対パスのsymlinkが含まれることからもそれらが他人の環境で動かないことがわかるかと思います。


## virtualenv

[pypa/virtualenv: Virtual Python Environment builder](https://github.com/pypa/virtualenv)

こちらは標準モジュールではないのでpipでインストールします

生成物見るとlib以下がsite-packages以外のファイルのsimlinkも持ってたり、pythonの実行ファイルもsimlinkじゃなかったりするようですが、使い方はほとんどvenvと同じです

```sh
$ pip install virtualenv
# virtualenv [your_env_name]
$ virtualenv test_env
$ source test_env/bin/activate
```

## pyenv-virtualenv

pyenvは仮想環境ではなく、複数バージョンのpythonを入れたい場合の管理ツールです。

pyenvは `.python-version` を見てpythonのバージョンを変えます。

更にpyenv自体がpythonインストールマネージャ的でもあるので、様々なversionのPythonを始め、Java実装であるJythonはもちろんanacondaまでもinstall可能です。

```sh
$ pyenv local system
$ python -V
Python 2.7.10 
$ cat .python-version
system

# 3.6.8をpyenvからinstallしたい場合
$ pyenv install 3.6.8
# anyenv経由でpyenv使ってるのでちょっとパスが特殊ですが利用イメージ
$ pyenv versions
  system
* 3.6.8 (set by /Users/yourusername/.anyenv/envs/pyenv/version)
  3.6.8/envs/sample_foxdot
  sample_foxdot
$ pyenv global 3.6.8
$ python -V
Python 3.6.8
```

pyenv-virtualenvはpyenvとvirtualenvを使ったwrapperです

pyenvに乗っかりつつvirtualenvできる感じ

```sh
$ pyenv virtualenvp 3.6.8 test_env
$ pyenv activate test_env
# PJディレクトリでこれでもOK
$ pyenv local test_env
```

やはり `.python-version` はgitに含めないようにしましょう

特筆すべきは
- venvやvirtualenvと違い、仮想環境ディレクトリの作成先が `pyenv/versions/[your_env] `配下
- 環境毎に名前をつける必要がある

の2点です。

例えばPyCharmでpythonの実行ファイルを指定する必要があるときはpyenv配下を漁りましょう。

また、ディレクトリの場所に依存しないので汎用的な環境を作っておくと使いまわせたりします。

## pipenv

pipenvはさらにpipをwrapし、npmのようなパッケージ管理の機能を追加したツールです

pyenvを入れていれば

```sh
# 色んなバージョン指定の方法
$ pipenv --three
$ pipenv --python 3.6
$ pipenv --python 3.6.8

# 仮想環境を有効化する
$ pipenv shell
```

仮想環境の位置ですが、 `PIPENV_VENV_IN_PROJECT`　の環境変数をtrueにしておくとカレントの.venv配下に作られます。defaultだとホームディレクトリの.virtualenv配下に作られます。
好みに応じて使い分けると良いと思います。

pipenvの利点と、他と大きく違うところはnpmライクなパッケージマネージャの機能があることです。

従来のpipによるパッケージ管理は `pip freeze` の結果を `requirements.txt` というファイルに書き込むという慣例で成り立っていました。

よって、複数人で開発している場合、pip install したファイルをrequirements.txtに追加し忘れるという問題が起こりえましたが、pipenvはこれを解決しています。

例えばパッケージはpipではなくpipenv経由でinstallします。

```sh
$ pipenv install requests
```

installしたパッケージは自動的にPipfileというファイルに書き込まれます。

```sh
$ cat Pipfile
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]

[packages]
requests = "*"

[requires]
python_version = "3.6" 
```

また、pip freezeではinstallしたパッケージの依存関係にあるパッケージまで出力するため、本当に必要として入れたパッケージが何なのか？わからなくなりがちです。

pipenvなら依存ファイルはPipfile.lockというファイルに書き込まれます(大きいため内容は載せません)。もちろんこれらも自動です。

更に開発環境でのみ必要なパッケージなどもrequirements.dev.txtのような独自管理だったと思いますがこれでも解決しています。

```sh
$ pipenv install --dev [pkg_name]
$ cat Pipfile
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]
[pkg_name]

[packages]
requests = "*"

[requires]
python_version = "3.6" 
```

便利！

### script

TODO

## Poetry

コマンド見る感じ使用感はpipenvっぽい？要調査

[sdispater/poetry: Python dependency management and packaging made easy\.](https://github.com/sdispater/poetry)



# まとめ

さまざまな仮想環境ツールがありますが、基本的には、

- PATHに仮想環境作成時のpythonのsymlinkを追加する
- sys.pathにsite-packagesを追加する

を実現するためのツール。

その他にも、

- pythonのバージョン管理も内包
- (pip以外の)pythonのパッケージ管理も内包

があるので必要に応じて使い分けましょう。

