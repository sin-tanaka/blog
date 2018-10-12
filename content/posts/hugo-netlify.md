---
title: "Hugo + Netlify でBlog作成"
date: 2018-10-12T12:28:52+09:00
draft: true
thumbnail: "/images/netlify.png"
toc: true
---

Golang製の静的サイトジェネレータHugoと最近良く名前を聞くホスティングサービスNetlifyで本ブログを作成したときのメモです。

## 参考

参考というか、まるっとそのままやってます。ありがとうございます。submoduleの部分だけ少し変えました。

[NetlifyでHugoで作った静的サイトをホスティングしてビルドを自動化する](https://blog.mismithportfolio.com/web/hugo-netlify-build)

## localでhostするまで

```sh
# install hugo
$ brew install hugo

# check version
$ hugo version
Hugo Static Site Generator v0.49/extended darwin/amd64 BuildDate: unknown

# create new site
$ hugo new site hogehoge-dot-com

$ cd hogehoge-dot-com

# set theme
$ cd theme
$ git clone https://github.com/dim0627/hugo_theme_robust.git

$ cd ../

# create new post
$ hugo new posts/my-first-post.md

# edit post
$ vim content/post/my-first-post.md

# serve on local
$ hugo server --buildDrafts --theme=hugo_theme_robust
```

## gitの設定

git管理下のディレクトリのthemeに対し更にgit cloneしているので、submoduleの設定をする。

また、今回リポジトリのホスティングにはGithubを使ったが、Netlifyはbitbucket / Gitlab も対応しているっぽい。

```sh
$ git init
$ git remote add origin https://github/<your-name>/<repo>.git
$ touch .gitmodules
$ vim .gitmodules

$ git add .
$ git commit -m "first commit"
$ git push origin master
```

`.gitmodules`

```yaml
[submodule "themes/hugo_theme_robust"]
    path = themes/hugo_theme_robust
    url = https://github.com/dim0627/hugo_theme_robust.git
```

## Netlify側でGithubへのmaster pushのタイミングでデプロイする設定をする

Netlify側の設定は、

1. githubと連携
1. ブログのリポジトリを選択
1. 静的サイトジェネレータにHugoを使う指定をする
1. Hugo用のデプロイのテンプレートを確認して問題なさそうなら完了

で終わりました。

ここはGUIなので難しくない。

参考までに設定済のDeploy Settingが以下のようになる。


![Netlify Deploy Setting](/images/hugo-deploy-setting.png)


これでGithubのmasterブランチへのpushをトリガーにしてhugoのビルドコマンドが走る。Netlify側ではビルドした生成物である `/public` ディレクトリをホストする。

著名な静的サイトジェネレータに特化し、めちゃくちゃ設定が簡単はCIといった感じ。とても便利。

