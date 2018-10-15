---
title: "PyCharmの機能をIdeaVimのnormalモードに割り当てる"
date: 2018-10-15T17:58:36+09:00
draft: true
thumbnail: "/images/pycharm.png"
toc: true
---

IDEにPyCharm、Vim用のキーバインド設定のためにIdeaVimというPluginを使っています。



PyCharmはそれ自体が便利で、機能も豊富で素晴らしいIDEです。今回はPyCharmに備わっている各機能をVimのnormalモードに割り当てるまでの設定をしたいと思います。

今回はPyCharmですが、JetBrains系のIDEであれば同様に設定できるかと思います。

---

今回は `Navigator > Next Method / Previous Method` をVimのnormalモードの `<C-n / C-p>` に割り当てます。

![NextMethod](/images/nextmethod.png)

`Next Method / Previous Method` は、名の通り、今いる位置から次/前のメソッドの行頭に移動する機能です。

デフォルトではCtrl + Shift + Option[Alt] + ↓ / ↑ なので結構押しづらい。

単に当該機能のキーマップを Ctrl + n / p に割り当てることでも問題ないのですが、自分の設定だと、それぞれUp / Downが割り当てられているため、normalモードのみこの機能が効くようにしたい。

## .ideavimrc を作成する

通常のVimを使用するときに、 `.vimrc` を設定するように、IdeaVimでも同様の設定ファイル `.ideavimrc` があります。

これに通常のVimの設定に加えて、IdeaVim独自の設定をします。

```shell
$ touch ~/.ideavimrc
```


## .ideavimrc を設定する

以下のようにnormalモードの <C-n/p> にそれぞれの機能をマッピングします。

```vim
nnoremap <C-n> :action MethodDown<CR>
nnoremap <C-p> :action MethodUp<CR>
```

なお、設定できるPyCharmのアクションの一覧は、IdeaVimを導入したPyCharmのエディタ上で、 `:actionlist` とすると見ること出来ます。

また、 `:actionlist method` とすることで、名称にmethod を含むアクションを見ることができます。


参考までに、私のideavimrcを掲載しておきます。

Alt + Enterに相当する ShowIntentionActions も Ctrl + Enterにマッピングしています。

```
vnoremap cc :action CommentByLineComment<CR>

nnoremap == :action ReformatCode<CR>
vnoremap == :action ReformatCode<CR>

nnoremap R :action RenameElement<CR>
nnoremap <C-CR> :action ShowIntentionActions<CR>

nnoremap cn :action GotoNextError<CR>
nnoremap cp :action GotoPreviousError<CR>

nnoremap <C-n> :action MethodDown<CR>
nnoremap <C-p> :action MethodUp<CR>

set clipboard+=unnamed
set surround
set hlsearch
set incsearch
set ignorecase
set smartcase
```

## 結果

![NextMethodResult](/images/mapping.gif)


## まとめ

ideavimrcを設定することでPyCharmの機能をVimのnormalモードに割り当てることが出来ました。

Vimを使う大きなメリットとして、モードという概念を扱えるという点があると思うので、うまく割り当てて、PyCharm + IdeaVimで効率よく作業を進めたいですね。

