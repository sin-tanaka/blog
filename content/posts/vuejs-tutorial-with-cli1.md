---
title: vue-cliで始めるVue.jsチュートリアル (1)
description: 本記事は vue-cliで始めるVue.jsチュートリアル (1) - Qiita を移植したものです
date: 2020-05-27T05:36:39.188Z
thumbnail: /images/uploads/download-1.png
categories:
  - Vuejs
---
本記事は [vue\-cliで始めるVue\.jsチュートリアル \(1\) \- Qiita](https://qiita.com/sin_tanaka/items/29769266b3b078ea0f7c) を移植したものです。

かなり古い内容ですが更新ができておらず、また2020年になってもvue-cli2.x系の記事にLGTMがついてしまうのは不味いのではないかと思い、Qiita側では文章を削除しております。

以下記事は上記を踏まえて参考にしていただければと思います。

以下転機

---

# Vue.js チュートリアル

最近Vue.jsが楽しいです。そこそこ構文にも慣れてきたので、自身の理解度チェックも兼ねてTodo管理アプリを作成するチュートリアルを作成してみました。間違っている箇所あればご指摘頂ければ幸いです。

**※2018.9.10 追記**

本記事執筆時は、`vue-cli` のバージョンは ｖ2.x.x 系でしたが、現在では v3.0.0 がリリースされていることにご注意ください。

https://github.com/vuejs/vue-cli/releases

テンプレートからプロジェクトを作成するときの対話形式が大きく異なっております。また、そのほかにも変更点があるかもしれません。折を見てアップデートする予定ですが、それまではv3.x.x系には未対応ということでお願いします。大筋は変わらないはず。。

## チュートリアルのゴール

* vue-cliをつかってプロジェクトを作成する
* Vue.jsでTodo管理アプリケーションを作成する
* 作成したアプリケーションをvue-loader、webpackでビルド／バンドルして、GitHub-pagesで配信してみる

### できたらやりたいこと

* コンポーネントに分割（親コンポーネント・子コンポーネント間でのデータのやり取り）
* Vuexの導入

### 詰んだ時

公式サイト:
https://jp.vuejs.org/index.html

日本語の公式ドキュメントが充実しているのは私達がVue.jsを使う上での利点の一つです。困った時は公式ドキュメントを読んでみるとよいです。

## 作るもの

以下のようなTodo管理アプリケーションを作ってみます。

* GitHub-pages: https://sin-tanaka.GitHub.io/vuejs_tutorial_todo_management/
* リポジトリ: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/

## 環境

```console
$ sw_vers
ProductName:    Mac OS X
ProductVersion: 10.11.6
BuildVersion:   15G1611

$ node -v
v6.11.3

$ npm -v
3.10.10
```
vue-cliを使うためにはnodeとnpmが必要です。
エディターはIntelliJ系のIDEであれば、Vue.js用のプラグインがあるのでオススメです。筆者はPycharmです。

## プロジェクトの作成

まずはvue-cliをinstallします。
vue-cliは雛形からプロジェクトを作成してくれる公式ツールです。公式には、「nodeやnpm、webpackに詳しくないならあまり使わないほうがいいよ」と書いてあるのですがとても便利なので使います。

```console
$ npm install -g vue-cli

$ vue --version
2.8.2
```

`vue init <template> <project-name>` でプロジェクトを作成します。
ここではwebpackというテンプレートを使い、tutorial_vuejs_todo_managementというプロジェクトを作成しています。


---

**※2018.5.7 追記**

以下のPRの変更によって、生成されるプロジェクトと、記事記載の内容とで差分があるのでご留意下さい（Hello.vue→HellowWorld.vueへのコンポーネントファイル名変更など）
基本的には最新のテンプレートを正とすることで問題ありません。

参考PR Apply few style guide rules #944
https://github.com/vuejs-templates/webpack/pull/944

公式スタイルガイド:
https://jp.vuejs.org/v2/style-guide/index.html

---

この時いくつか質問されます。私はLinterと単体テストとe2eテストのツールは外していますが、全てYesでも良いです。
公式ではESLintを使うことが推奨されています。しかし、jsコーディング規約に慣れていない人には結構つらいです。これを機会に慣れるのも有りだと思います。
Linterを使う人は、通常のLinterか、airbnbのLinterを選べますのでお好きな方を選びましょう。
ここでは初学者向けのチュートリアルということでLinterは外しています。

```console
$ vue init webpack tutorial_vuejs_todo_management                                       

? Project name tutorial_vuejs_todo_management
? Project description A Vue.js project
? Author Shintaro Tanaka
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? No
? Setup unit tests with Karma + Mocha? No
? Setup e2e tests with Nightwatch? No

   vue-cli · Generated "tutorial_vuejs_todo_management".

   To get started:

     cd tutorial_vuejs_todo_management
     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.GitHub.io/webpack
```


## ローカルサーバでホストする

`vue init` を実行したディレクトリにプロジェクトが作成されたので、`get started` の通りにコマンドを実行してみます。

```console
$ cd tutorial_vuejs_todo_management
$ npm install  # 少し時間かかる
$ npm run dev
```

上手く行けば`localhost:8080` でブラウザが開いて以下の画面が表示されるはずです。

![初期画面](https://qiita-image-store.s3.amazonaws.com/0/67790/66d0a470-53d1-c638-023b-10236b040d6f.png)


のちにGitHub-pagesを利用するため、GitHubを使います。
この段階でGitHubリポジトリの作成、git init、pushまでしてしまうと楽だと思います。

## ファイルの構成

作成したプロジェクトのファイル構成を確認してみます。

主にかかわるファイルは以下になります。他はほとんど設定ファイルです。
筆者も設定ファイルについてはあまりよく分かっていないのですが、それでも動くものが作れてしまうのがvue-cliを使う大きなメリットだと思います。

```console
# 一部省略
tutorial_vuejs_todo_management$ tree
./index.html
./src
├── App.vue
├── assets
│   └── logo.png
├── components
│   └── Hello.vue
├── main.js
└── router
    └── index.js
```


一つずつザックリみて、全体の流れを把握してみます。

### index.html

```html:index.html

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>tutorial_vuejs_todo_management</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

これがルートのファイルっぽいですね。
cssもjsも読み込んでいませんが、最終的にはWebpackなどで一つのscriptファイルにバンドルされます。

`<div id="app"></div>` を覚えておいて下さい。

### src/main.js

```js:src/main.js
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App }
})
```

ここではVueインスタンスを作成して、Appコンポーネントをindex.htmlのid=appの要素に紐付けています。
また、importでApp、routerというモジュールを読み込んでいるようです。

### src/App.vue

```html:src/App.vue
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

先程読み込んでいたAppの実体です。`*.vue`という拡張子ですが、これはVueコンポーネントを記述するものです。
vue-loaderというモジュールでブラウザで読み込める形へコンパイルされます。
ここではAppコンポーネントを定義しています。
Helloコンポーネントの説明のときに詳しく説明しますので、ここではザックリ中身を見てみます。

<template>の中身を見ると、画面のVueのロゴはAppコンポーネントで出力しているようです。
又、`<img>` タグ下の`<router-view>` というタグが気になりますね。

### src/router/index.js

```js:src/router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import Hello from '@/components/Hello'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello
    }
  ]
})
```

`<router-view>` の動作は`vue-router.Router` を読み込んだ`src/router/index.js`に定義されています。`vue-router` はルーティングと、それに対応して出力するコンポーネントを決めています。
ここでは`/` にアクセスした時、Helloコンポーネントを出力するように設定しています。ルーティングを追加するのは簡単で、routesの配列にオブジェクトを追加していくだけです。
ここではHogeコンポーネントがあると仮定し、`/hoge` にアクセスした時Hogeコンポーネントを返すルーティングを設定する例を示します。

```js:src/router/index.jsにルーティングを追加した例
export default new Router({
  routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello
    },
    {
      path: '/hoge',
      name: 'Hoge',
      component: Hoge
    }
  ]
})
```

### src/components/Hello.vue

`router/index.js` ではルートにアクセスしたとき、Helloコンポーネントを出力していることが分かりました。
Helloコンポーネントを見てみます。

```html:src/components/Hello.vue
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
      <li><a href="https://vuejs.org" target="_blank">Core Docs</a></li>
      <li><a href="https://forum.vuejs.org" target="_blank">Forum</a></li>
      <li><a href="https://chat.vuejs.org" target="_blank">Community Chat</a></li>
      <li><a href="https://twitter.com/vuejs" target="_blank">Twitter</a></li>
      <br>
      <li><a href="http://vuejs-templates.GitHub.io/webpack/" target="_blank">Docs for This Template</a></li>
    </ul>
    <h2>Ecosystem</h2>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://GitHub.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
```

少々長いので、3つに分割してみます。

```html
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
      <li><a href="https://vuejs.org" target="_blank">Core Docs</a></li>
      <li><a href="https://forum.vuejs.org" target="_blank">Forum</a></li>
      <li><a href="https://chat.vuejs.org" target="_blank">Community Chat</a></li>
      <li><a href="https://twitter.com/vuejs" target="_blank">Twitter</a></li>
      <br>
      <li><a href="http://vuejs-templates.GitHub.io/webpack/" target="_blank">Docs for This Template</a></li>
    </ul>
    <h2>Ecosystem</h2>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://GitHub.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
  </div>
</template>
```

画面下部のリンクはこの部分に記述されているようです。`{{ msg }}` や`<template>` を除けば普通のhtmlですね。

```html
<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
}
</script>
```
`<script>` タグで囲われているのでjsっぽいですね。上で出てきた`{{ msg }}` もここで定義されている感じです。

```html
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
```

ここも`<style>` タグで囲われているので普通のcssっぽいですね。`scoped` というプロパティが気になるくらいでしょうか。

## Vueコンポーネントファイルでしていること

ひと通り見終えたので、このコンポーネントで行っているであろうことをまとめてみます。

* `<template>` にhtml構造の記述
* `<script>` にjsを記述　html中に書かれているmsgもここで定義
* `<style>` にcssを記述

上記の3点をひとまとめにして`*.vue` というファイルとしているようです。

html、js、cssは分けて記載するのが一般的ですが、コンポーネントという考えでは、それらをまとめて記述することで、再利用性や、見通しを良くしています。
責務の分担という意味ではオブジェクト指向的でもあります。

Vueコンポーネントの詳細は以下のドキュメントを参照下さい。

[コンポーネント](https://jp.vuejs.org/v2/guide/components.html)
[Vue Component の仕様](https://vue-loader.vuejs.org/ja/start/spec.html)

ざっくり解説すると、

`<template>`タグは文字列に展開され、Vueコンポーネントのtemplateオプションに渡されます。

また、`<style>` タグでは`scoped` を指定することでscoped cssを実現しています。この`<style>` タグに書かれたCSSは、このコンポーネントの中でのみ適用されます。
なのでBEMほどカッチリとしたCSSを書かなくてもOKです（ただし一貫性は持ったほうが良いと思いますし、タグ指定よりclassやid指定のほうが速いです）

[スコープ付き CSS](https://vue-loader.vuejs.org/ja/features/scoped-css.html)


`<script>`タグではVueコンポーネントのオプションのオブジェクトをエクスポートします。

```js
Vue.component('my-component',{
  // オプション
})
```

ここではVueコンポーネントに渡す引数として、dataを渡しています。
このときdataは
* 関数であること
* コンポーネントで扱いたいデータをオブジェクトに定義し、returnする

ことで定義したデータは`<template>` の中で`{{ }}`を囲うことで出力することができます。

画面に出力されている`Welcome to Your Vue.js App` はVueインスタンスの中に定義されたmsgを出力していることがわかります。

### dataオプションの書き方

dataオプションは以下のように書くことも可能です。

```js
// OK
data: function () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
```

このときアロー関数を使わないようにしましょう、変数のスコープが変わってしまうため推奨されません。

[インスタンス内において、アロー関数の「this」はインスタンスを参照しない](http://nayucolony.hatenablog.com/entry/2017/05/31/232024)

```js
// NG
data: () => {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
```

ここまでで全体の流れの説明は終わりです。
次回からはこれらのファイルへ追加／修正することで、Todoリストを作ってみます。
