---
title: Pythonの仮想環境構築についてまとめ【社内向け】
description: 2019-11-11時点で知ってることについてまとめておこうと思います。有識者によるツッコミ歓迎です。
date: 2019-11-11T00:38:09.000Z
thumbnail: /images/uploads/download.png
categories:
  - python
  - virtualenv
  - pyenv
  - venv
  - pipenv
---

2019-11-11時点で知ってることについてまとめておこうと思います。

## 前提

- 状況に応じて使い分けましょう
  - 2/3系の最新、ぐらいの大きなくくりでも問題ない人
  - 2.6環境を用意しなければいけないとき😔
  - Win環境でデータ解析した人で仮想環境とか考えなくてもいいのであれば全部入りのanacondaでもいいかもしれない
    - 標準でPythonが入っているLinux環境(Mac含む)では推奨しない
    - (仮想環境を使いこなせばanacondaのような環境作るは簡単)
- 良質な仮想環境構築記事を見分けられるようになりましょう
- お前を信じる仮想環境を信じましょう

## 最初にまとめ

さまざまな仮想環境ツールがありますが、基本的には、

- PATHに仮想環境作成時のpythonのsymlink or 実体を追加する
- sys.pathにsite-packagesを追加する

を実現するためのツールなので必要に応じて使い分けましょう。

その他にも、

- pythonのバージョン管理も内包
- (pip以外の)pythonのパッケージ管理も内包

があるので必要に応じて使い分けましょう。

筆者は以下を必要に応じて使い分けてるので、いずれにせよ `pyenv` はあると楽かも

- pyenv + venv
- pyenv-virtualenv
- pyenv + pipenv

## venv

公式の提供するモジュールです。3.3からデフォルトで使えます。

[venv \-\-\- 仮想環境の作成 — Python 3\.8\.0 ドキュメント](https://docs.python.org/ja/3/library/venv.html)

※ 2.7等には入ってません

## `仮想環境を有効化する` とは

venvでの仮想環境構築を通しておおまかな仮想環境有効化の動きを追ってみます。

```sh
$ python -V  # Python 2.7.10
$ python3 -V  # Python 3.6.8
$ pwd
/Users/yourusername/foobar

# python3 -m venv /path/to/new
$ python3 -m venv ./env
```

とすると `/path/to/new` なディレクトが作られます(ここでは `env` )。これが仮想環境のディレクトリになります。

treeコマンドで中身を見てみます。

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
```

`./env/bin` 以下に、venv実行時のpythonの実体に向いた `symlink`、や `pip` 、 `activate` 等があるのがわかります。

次に `./env/bin` にある `activate` を実行し、仮想環境を有効化してみます。

`activate` ですが、OSはもちろん、shellごとにも異なったりするので(たとえば `.fish` とか `.csh` があるのが見えると思います)、公式ドキュメント参照のこと。

```sh
$ source ./env/bin/activate
```

さて何が変わったのでしょうか

```sh
env > $ python -V  # Python 3.6.8
```

まず、 `python` コマンド実行時のバージョンが `python3` 実行時のバージョンである `3.6.8` になっています。

仮想環境有効化の前後のPATHをみてみます。

```sh 
env > $ echo $PATH
/Users/yourusername/foobar/env/bin:/usr/local/Cellar/zplug/2.4.2/bin:・・・

env > $ deactivate
/usr/local/Cellar/zplug/2.4.2/bin:・・・
```

仮想環境を有効化することで./env/binがPATHに追加されています。

これによって `python` 実行で参照しているバージョンが3.6.8になっているということがわかります。

---

さらに外部のパッケージの参照パスについて見てみます。

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

基本的に、仮想環境下では、元のpythonで入れたパッケージは参照することができません。

よって、仮想環境内のpipを通してパッケージをインストールすることで仮想環境内でパッケージをimportできるようになります。

```sh
$ source env/bin/activate
env > $ pip install requests
env > $ python 
>>> import requests
```

こちらも仮想環境の有効化前後で `sys.path` を比較してみます

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

---

以上が `「仮想環境を有効化する」` と表現していることの実体です。

`venv` はシンプルな動きですし、プロジェクトのディレクトリ内に仮想環境を持てるというのは意味的にスッキリしています。基本的な仮想環境の動きを理解するのに最適かもしれません。

例えば、後述する `pyenv-virtualenv` 等は単一の名前空間に仮想環境を作るようなイメージなので、プロジェクトとの関連が薄いです。

注意点として、venvで作成した仮想環境ディレクトリはgit管理に含めないようにしましょう。絶対パスのsymlinkが含まれることからもそれらが他人の環境で動かないことがわかるかと思います。

## virtualenv

[pypa/virtualenv: Virtual Python Environment builder](https://github.com/pypa/virtualenv)

こちらもシンプルです。

venvと生成物比較するとlib以下がsite-packages以外のファイルのsimlinkも持ってたり、pythonの実行ファイルもsimlinkでなく実体なので容量食ったりですが、使い方はほとんどvenvと同じです

```sh
$ pip install virtualenv
# virtualenv /path/to/new
$ virtualenv ./env
$ source ./env/bin/activate
```

## pyenv-virtualenv

`pyenv` は複数バージョンのpythonを入れたい場合のパッケージマネジャー的なツールです。

様々なversionのPythonを始め、Java実装であるJythonはもちろんanacondaまでもinstall可能です。

`.python-version` というファイルを見てpythonのバージョンを変えます。

```sh
$ pyenv local system
$ python -V
Python 2.7.10 

$ cat .python-version
system

# 3.6.8をpyenvからinstallしたい場合
$ pyenv install 3.7.0

# anyenv経由でpyenv使ってるのでちょっとパスが特殊ですが利用イメージ
$ pyenv versions
* system
  3.7.0 (set by /Users/yourusername/.anyenv/envs/pyenv/version)

$ pyenv global 3.7.0
$ python -V
Python 3.7.0

$ pyenv versions
  system
* 3.7.0 (set by /Users/yourusername/.anyenv/envs/pyenv/version)
```

---

`pyenv-virtualenv` は `pyenv` と `virtualenv` を使ったwrapperです

`pyenv` に乗っかりつつ `virtualenv` できる感じです

```sh
# インストールしたバージョンから仮想環境を作る
$ pyenv virtualenv 3.7.0 3.7.0-env

$ pyenv versions
  system
  3.7.0 (set by /Users/yourusername/.anyenv/envs/pyenv/version)
  3.7.0/envs/3.7.0-env
* 3.7.0-env

# 有効化する
$ pyenv activate 3.7.0-env

# プロジェクトディレクトリでこれでもOK
$ pyenv local 3.7.0-env
```

やはり `.python-version` はgitに含めないようにしましょう

`venv` や `virtualenv` との違いは、

- 仮想環境ディレクトリの作成先が `$HOME/.pyenv/versions/[your_env] `配下
- 環境毎に名前をつける必要がある

の2点です。

例えばPyCharmでインタプリタのパスを指定するとき、 `virtualenv` 、 `venv` は勝手に読んでくれたりしますが、 `pyenv-virtualenv` の場合 `.pyenv` 配下を漁る必要があり少し面倒です。

また、仮想環境ディレクトリはディレクトリの場所に依存しないので汎用的な環境を作っておくと使いまわせたりします。

## pipenv

`pipenv` は `pip` をwrapし、 `npm` のようなパッケージ管理の機能 + `virtualenv` or `venv` を一元管理するツールです。

[Pipenv: 人間のためのPython開発ワークフロー — pipenv 2018\.11\.27\.dev0 ドキュメント](https://pipenv-ja.readthedocs.io/ja/translate-ja/)

公式も推奨しています。

PyCon 2018の作者の講演が参考になります

[Kenneth Reitz \- Pipenv: The Future of Python Dependency Management \- PyCon 2018 \- YouTube](https://www.youtube.com/watch?v=GBQAKldqgZs)

氏が問題点として上げているのが、

- requirements.txtによるパッケージ管理はlockfileの機構がない
- 依存関係が1ファイルにすべて保存されているため、本当に必要とされているパッケージがわからない

という点です。

例えば、 `pip install flask` としたとき、真にプロジェクトに必要なパッケージは `flask` のみであるはずが、`flask` の依存パッケージである `click` や `jinja2` までもを `requirements.txt` で管理する必要があります。

これを `pipenv` では必要なパッケージは `Pipfile` というファイルに、その依存関係を `Pipfile.lock` と分けることでこれを解決しています。

また、開発でのみ必要なパッケージ -例えば `pytest` など- の管理も可能です。

`virtualenv` によって解決したい領域とは違うため、「仮想環境構築ツールとして語るのはどうなの？」と思うかもしれませんが、 `pipenv` はとにかく多機能で、仮想環境作成の機能や、npmライクなscript管理も可能です(仮想環境構築、という点でいうとオーバースペックかも)

仮想環境の構築には `virtualenv` or `venv` を使うため、それらの動きをすでに理解している方や、 `requirements.txt` によるパッケージ管理に問題を抱えている方には有用かと思います。

```sh
# 仮想環境を作成
# 色んなバージョン指定の方法
# このときpyenvが入っていると、指定したバージョンが未インストールでもインストール可能
$ pipenv --three
$ pipenv --python 3.6
$ pipenv --python 3.6.8

# 仮想環境を有効化する
$ pipenv shell
```

仮想環境の位置ですが、デフォルトではホームディレクトリの.local配下に作られます。`PIPENV_VENV_IN_PROJECT`　の環境変数をtrueにしておくとカレントディレクトリのの.venv配下に作られます。

参考: [Pipenvと仮想環境 — pipenv 2018\.11\.27\.dev0 ドキュメント](https://pipenv-ja.readthedocs.io/ja/translate-ja/install.html#virtualenv-mapping-caveat)

--- 

パッケージのinstallはpipenv経由で行います。このとき自動的にPipfile / Pipfile.lockへ書き込まれます。

```sh
$ pipenv install flask
```

```toml
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]

[packages]
flask = "*"

[requires]
python_version = "3.6" 
```

開発用のパッケージも同様です `--dev or -d` をつけるだけです

```sh
$ pipenv install --dev pytest 
```

```toml
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]
pytes = "*"

[packages]
flask = "*"

[requires]
python_version = "3.6" 
```

Pipfile / Piplock.fileはgit管理に含めましょう。Pipfileがあるディレクトリで `pipenv install` とすると、Pipfileに沿ったバージョンの仮想環境とパッケージのインストールが可能です。ここもnpmライクですね。

### script管理

[Pipenvの進んだ使い方 — pipenv 2018\.11\.27\.dev0 ドキュメント | 独自のスクリプトショートカット](https://pipenv-ja.readthedocs.io/ja/translate-ja/advanced.html#custom-script-shortcuts)

開発をしているとサーバの起動コマンドが複数必要なケースがあると思います。

例えばDjangoでrunserverするときに参照するsettings.pyを変えたいときなどが考えられますが、コマンドが長くなりがちですし、READMEで管理するのも面倒かと思います。

こういったスクリプトをPipenvのscriptsに登録することで、コマンドの抽象化ができ、プロジェクトで必要なコマンドが一覧で分かります。

```sh
# pipenv run [script_name]
$ pipenv run start
```

```toml
[scripts]
start = "python manage.py runserver --settings config.setting.dev"
format = "black ."
```

scriptで実行されるpythonはｍ仮想環境を有効化しているかに関わらず、仮想環境下で実行されます。

また `.env` というファイルをプロジェクトに作成すると `pipenv run` `pipenv shell` 時に自動で読み込みます。

[Pipenvの進んだ使い方 — pipenv 2018\.11\.27\.dev0 ドキュメント | .env の自動読み込み](https://pipenv-ja.readthedocs.io/ja/translate-ja/advanced.html#automatic-loading-of-env)

## Poetry

コマンド見る感じ使用感はpipenvっぽい？要調査

[sdispater/poetry: Python dependency management and packaging made easy\.](https://github.com/sdispater/poetry)

## まとめ

さまざまな仮想環境ツールがありますが、基本的には、

- PATHに仮想環境作成時のpythonのsymlinkを追加する
- sys.pathにsite-packagesを追加する

を実現するためのツール。

その他にも、

- pythonのバージョン管理も内包
- (pip以外の)pythonのパッケージ管理も内包

があるので必要に応じて使い分けましょう。
