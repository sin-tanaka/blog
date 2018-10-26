---
title: "一日のcommitを振り替える - git today"
date: 2018-10-26T19:21:57+09:00
draft: true
thumbnail: "/images/git_logo.png"
toc: true
---

独自にgit.config.aliasに定義している `git today` というコマンドを紹介します。

# git today

git管理下のディレクトリ上で `git today` とすると今日一日のコミットログが出力されます。

`~/.gitconfig`

```
[alias]
    today = "!f () {\
                 git log --oneline --branches \
                 --since=midnight --date=iso \
                 --author=\"$(git config user.name)\" \
                 --format=\"- %s\";\
             };f"
```

```bash
$ git today
- add hoge fuga
- とりあえずcommit
```

gitをCUIから使うのであればalias活用していきたいですね。

参考までに私のgitconfigです。

https://github.com/sin-tanaka/dotfiles/blob/master/.gitconfig


## 参考

[Qiita - "git log" で今日行った作業を表示&ファイル出力するときの注意](https://qiita.com/sgr-ksmt/items/65ddde68173dab9a98e9)
