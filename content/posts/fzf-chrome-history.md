---
title: fzf + sqliteを使ってChromeの閲覧履歴をインクリメンタルサーチする
description: fzf + sqliteを使ってChromeの閲覧履歴をインクリメンタルサーチする
date: 2018-12-12T03:00:12.697Z
thumbnail: /images/uploads/スクリーンショット-2018-12-12-10.21.20.png
categories:
  - fzf
  - sqlite
  - 開発効率
---
[JSL \(日本システム技研\) Advent Calendar 2018 \- Qiita](https://qiita.com/advent-calendar/2018/jsl) の12日目の記事です。

## モチベーション

- Chromeの履歴が見辛いのでいい感じに表示したい
- 履歴を見やすくするプラグインはいくつかあるが、ターミナルから「履歴閲覧→開く」ができるとシームレスにWebブラウジングに移れそう

なので今回は、sqliteを使ってユーザのローカルに保存してある履歴ファイルを閲覧 + fzfを使ってインクリメンタルサーチ　するコマンドを実現するための方法を書く。

## 最終的にやったこと

fzfの公式サンプルに載ってた。

[Examples · junegunn/fzf Wiki](https://github.com/junegunn/fzf/wiki/examples#google-chrome-os-xlinux)

サンプルに沿って、パスの通った場所に以下のようなコマンドを置きました。名前はChromeHistoryで `ch` としました。

```zsh:ch
#!/usr/bin/env zsh

ch() {
  local cols sep google_history open
  cols=$(( COLUMNS / 3 ))
  sep='{::}'

  if [ "$(uname)" = "Darwin" ]; then
    google_history="$HOME/Library/Application Support/Google/Chrome/Default/History"
    open=open
  else
    google_history="$HOME/.config/google-chrome/Default/History"
    open=xdg-open
  fi
  cp -f "$google_history" /tmp/h
  sqlite3 -separator $sep /tmp/h \
    "select substr(title, 1, $cols), url
     from urls order by last_visit_time desc" |
  awk -F $sep '{printf "%-'$cols's  \x1b[36m%s\x1b[m\n", $1, $2}' |
  fzf --ansi --multi --preview-window down:1 | sed 's#.*\(https*://\)#\1#' | xargs $open > /dev/null 2> /dev/null
}

ch $@
```

以下のように動作する。

![fzfとsqliteを組み合わせてChromeの履歴を閲覧](/images/uploads/history.gif "history.gif")

### 解説

以下簡単な解説。


### fzf

fzfはターミナル上であいまい検索を実現するコマンドです。

[junegunn/fzf: A command\-line fuzzy finder](https://github.com/junegunn/fzf)

簡単な使い方としては以下のように標準出力をパイプを通してfzfに渡すとあいまい検索をしてくれる。

```bash
$ find . | fzf
```

セットアップ時によく使うコマンドをキーバインドとして提供してくれるのが便利。

[junegunn/fzf: A command\-line fuzzy finder#key-bindings-for-command-line](https://github.com/junegunn/fzf#key-bindings-for-command-line)

`CTRL + R` でhistoryコマンドの実行結果をfzfに渡したコマンドを実行してくれるし、 `CTRL + T` でカレンドティレクトリ以下のファイルとディレクトリを検索した結果をfzfに渡したコマンドを実行してくれる。


また、fzfで選択した結果は更に標準出力されるのでパイプや `xargs` を使えばその用途が多岐に渡るのが想像できると思う。

例えば、プロセスIDの検索結果を `kill` コマンドに渡してあげるとか、gitブランチの検索結果をgit checkoutに渡してあげるとか、そういった用途に便利に使うことができる。


以下の記事がとてもわかり易いです。

[fzf（fuzzy finder）の便利な使い方をREADME, Wikiを読んで学ぶ \- もた日記](https://wonderwall.hatenablog.com/entry/2017/10/06/063000)

---

類似のツールには `peco` というものがありますが、どちらも実現できることを一緒なので好きな方を使えばいいと思う。両者ともに日本語の知見は豊富。


### Chromeの履歴ファイルの場所

執筆時のChromeのバージョンは以下。

- バージョン: 70.0.3538.110（Official Build） （64 ビット）

--- 

Chromeの履歴を見る方法は3つぐらいあって、

1. `CMD + Y`
2. Chromeの検索バーに `chrome://history` と入力する
3. 右上の三点リータのボタンから履歴 > 履歴

1が最も簡単だと思う。


ただこの履歴ページ、日付検索ができないという欠陥がありとても不便である。

まともに履歴を見ようとするならば、

- Chromeのプラグインを入れる このへんとか [Chrome Better History \- Chrome ウェブストア](https://chrome.google.com/webstore/detail/chrome-better-history/aadbaagbanfijdnflkhepgjmhlpppbad)
- 履歴ファイルはローカルに保存されているので、sqliteで読む [Chromeの閲覧履歴を取得する。 \- mirandora\.commirandora\.com](http://www.mirandora.com/?p=697)

のようにハックをすると便利。

OS Xの場合履歴ファイルの場所が以下となる。

```
$HOME/Library/Application Support/Google/Chrome/Default/History
```


公式サンプルではコマンド実行の都度、 `/tmp/` に履歴ファイルをコピーしsqliteで読む→awkで整形→fzfで検索 のような流れとなっている。

## まとめ

fzfを使ってChromeの履歴をインクリメンタルサーチできるコマンドを定義しました。

やはりCUIからなんでもできるのは便利です。

GUIでなくCUIを使うメリットはCUIだからとしか言えないのですが、今後も不便だと感じるところについては効率化していきたいですね。

