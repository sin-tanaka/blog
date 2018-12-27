---
title: コマンド履歴を見ながら今年便利だったコマンドを振り返る
description: コマンド履歴を見ながら今年便利だったコマンドを振り返る
date: 2018-12-27T09:53:20.923Z
thumbnail: /images/uploads/スクリーンショット-2018-12-27-19.01.17.png
categories:
  - 開発効率
  - fzf
  - git
  - tig
---
こんな感じの集計コマンドを用意しました。

```sh
# historyの内容によっては export LANG=C する必要がある
$ cat ~/.zsh_history | sort | uniq -c | sort -nr | less
```

上記コマンドの結果を見ながら、今年よく使ったコマンドを振り返ってみます。


inspired by [今年お世話になったCLIコマンド集 \- mizchi's blog](https://mizchi.hatenablog.com/entry/2018/12/21/114044)

## git branch -a | fzf | xargs git checkout

インタラクティブにgit checkoutするコマンドです。詳細はQiitaに書いてます。

[fzfでインタラクティブにgit checkout する \- Qiita](https://qiita.com/sin_tanaka/items/ca0fdd899e40a1d6716b)

ダントツで実行回数が多かったのでショートカットキーで起動できるようにしたほうがいい気がしてきた。

## cd

これは `enhanced` のaliasです。

[b4b4r07/enhancd: A next\-generation cd command with an interactive filter](https://github.com/b4b4r07/enhancd)

ターミナル上において大半の操作がディレクトリ移動っていう人は導入すると良いと思います。

## tig

便利な `git log --oneline` として重宝しました。また、 `tig stash` でstashをいい感じに見てました。

[jonas/tig: Text\-mode interface for git](https://github.com/jonas/tig)

他にも変更をstageに追加する操作で使ってます。

## git today

一日の git commmit を振り返るコマンドです。ブログに書きました。

[一日のcommitを振り替える \- git today \- sin\-tanaka\.com](https://blog.sin-tanaka.com/posts/181026-gittoday/)

最近は `git today | pbcopy` のように使うことが多いです。

## sudo networksetup -setsocksfirewallproxystate Wi-Fi [on|off]

MacOSでWiFiのネットワークインターフェースで `socks_proxy` を on / off するコマンドです。

システム環境設定からでも出来ますが、面倒なのでコマンド実行して切り替えています。それぞれ `socks_on / socks_off` のaliasを張ってます。

`socks_proxy` についてはあまり良くわかってませんが、リモート作業時にsshと組み合わせて便利に使っています。

## ipconfig getifaddr en0 | tr -d '\n'

en0のローカルIPを表示するコマンドです。 このコマンドも `ip` というaliasを張っていて、結果を `pbcopy` に渡したものを `ipp` としています。

## open .

カレントディレクトリをFinderで開くコマンドです。

他にもCordovaなどで都度生成されるxcodeprojを、 `open /Applications/Xcode.app /path/to/hoge.xcodeproj` のようにコマンドから立ち上げる用途でも使っていました。

## charm .

カレントディレクトリをPyCharmで開くコマンドです。去年Qiitaに書きました。

[PyCharmをコマンドラインから起動したい！ \- Qiita](https://qiita.com/sin_tanaka/items/2f6e6a891958b83cb69b)

PyCharm起動済だと開かないのが難点

## まとめ

上位のコマンドについてはほとんどalias化されていたのでうまく効率化出来ているなぁという印象でした。

また、git関連のコマンドが圧倒的に多かったです。
ブランチ切り替えについてはショートカットキーで起動できるように対応して、git statusは.gitがあるときは何もせずとも表示されるようにすれば良い気がしてます。

効率化は単に時短になるだけでなく、副作用として得られる生の知見も多いです（実行後、不可逆なコマンドにalias張ってミスした結果、重要なコマンドには張らなくなった等）。来年も積極的に進めていきたいです。
