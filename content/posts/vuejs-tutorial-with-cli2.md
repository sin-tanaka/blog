---
title: vue-cliで始めるVue.jsチュートリアル (2)
description: 本記事は vue-cliで始めるVue.jsチュートリアル (2) - Qiita を移植したものです
date: 2020-05-27T05:44:38.403Z
thumbnail: /images/uploads/download-1.png
tags:
  - Vuejs
---
本記事は [vue\-cliで始めるVue\.jsチュートリアル \(2\) \- Qiita](https://qiita.com/sin_tanaka/items/6d5d9089eb76dda4ce88) を移植したものです。

かなり古い内容ですが更新ができておらず、また2020年になってもvue-cli2.x系の記事にLGTMがついてしまうのは不味いのではないかと思い、Qiita側では文章を削除しております。

以下記事は上記を踏まえて参考にしていただければと思います。

以下転機

---

# Vue.js チュートリアル(2)

前回プロジェクトの作成と、その中身を見てみました。

[vue-cliで始めるVue.jsチュートリアル (1)](https://qiita.com/sin_tanaka/items/29769266b3b078ea0f7c)

ここからはこれらのコンポーネントを修正して、実際にTodoリストを作ってみます。

## Todoリストの要件
Todoリストアプリケーションの要件は以下のように定義しておきます。

1. Todoはリストで一覧表示すること
* Todoはテキストボックスから追加できること
* それぞれのTodoにはチェックボックスが付いており、それを切り替えることでTodoの状態（未達成／達成済）を切り替えること
* チェック済のTodoを一括で消すボタンがあること
* それぞれのTodoは編集可能なこと

一般的なCRUD(Create, Read, Update, Delete)を持つインターフェースだと思います。

最終的にできあがったTodoリストは`GitHub-pages`を使って配信するところまでを一先ずの目標とし、その後可能であれば、

* コンポーネントの分割（親子間でのデータのやり取り）
* Vuexの導入

まで出来れば理想ですが一先ず一つのコンポーネントにべた書きでTodoリストを作ってみましょう。

### SASS/SCSS

その前に、`*.vue` ファイル内の`<style>` タグ内で、`SASS/SCSS` を書けるようにしましょう（これは好みなので、普通のCSSでいい人は入れなくてもよいです。但しサンプルコードはSCSSで書かれています）
プロジェクトディレクトリ内で以下を実行します。

```bash
npm install sass-loader node-sass --save-dev
```

これでSCSSが書けるようになりました。

## HTML & CSSの作成

まずはhtmlとCSSでTodoリストのイメージを組み上げてみます。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/07faa150878b8dade8fa48ee4f58168da31d08a2

```html:src/App.vue
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1>Todo Management.</h1>
    <hr />
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

```html:src/components/Hello.vue
<template>
  <div>
    {{ msg }}
    <form>
      <button>ADD TASK</button>
      <button>DELETE FINISHED TASKS</button>
      <p>input: <input type="text"></p>
      <p>task:</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"><input type="checkbox"><button>EDIT</button>vue-router</label>
      <label class="task-list__item"><input type="checkbox"><button>EDIT</button>vuex</label>
      <label class="task-list__item"><input type="checkbox"><button>EDIT</button>vue-loader</label>
      <label class="task-list__item--checked"><input type="checkbox" checked><button>EDIT</button>awesome-vue</label>
    </div>
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
<style lang="scss" scoped>
@mixin flex-vender() {
  display: flex;
  display: -webkit-flex;
  display: -moz-flex;
  display: -ms-flex;
  display: -o-flex;
}
.task-list {
  @include flex-vender;
  flex-direction: column;
  align-items: center;
  &__item {
    width: 270px;
    text-align: left;
    $element: #{&};
    &--checked {
      @extend #{$element};
      color: #85a6c6;
    }
  }
}
</style>
```

以下のような画面になるはずです。このとき、`npm run dev` は起動しっぱなしでOKです。ソースを編集すると自動でコンパイル・リロードまでしてくれることが確認できると思います（ホットリロード）。

![02.png](https://qiita-image-store.s3.amazonaws.com/0/67790/89e73b1e-f4a9-5306-c656-69014356a813.png)

Todoのテキストは初期画面のテキストをそのまま使っています。各自変えてもらって問題ないです。
htmlとcssに手を加えただけなので、このままでは何も動作しません。

## リストレンダリング

次に、ボタンやテキストエリアに動作やデータを紐付けていきます。
まずは、`src/components/Hello.vue` で繰り返し出現しているTodoの一覧表示を`v-for` ディレクティブを使ってリストレンダリングしてみます。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/852419626e620efa0397f685e67f79b2ee926998
 
```html:src/components/Hello.vue
<template>
  <div>
    {{ msg }}
    <form>
      <button>ADD TASK</button>
      <button>DELETE FINISHED TASKS</button>
      <p>input: <input type="text"></p>
      <p>task:</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"
             v-for="todo in todos">
        <input type="checkbox"><button>EDIT</button>{{ todo.text }}
      </label>
    </div>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      todos : [
        {text : 'vue-router', done: false},
        {text : 'vuex', done: false},
        {text : 'vue-loader', done: false},
        {text : 'awesome-vue', done: true },
      ]
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
@mixin flex-vender() {
  display: flex;
  display: -webkit-flex;
  display: -moz-flex;
  display: -ms-flex;
  display: -o-flex;
}
.task-list {
  @include flex-vender;
  flex-direction: column;
  align-items: center;
  &__item {
    width: 270px;
    text-align: left;
    $element: #{&};
    &--checked {
      @extend #{$element};
      color: #85a6c6;
    }
  }
}
</style>
```

`<template>` の中で繰り返し表れていた`<label>` に`v-for` が追加され、`<template>` の中身がスッキリしました。

また、Todoの内容は`<script>` タグ内のdataオプションに移動しています。

解説をすると、`v-for="todo in todos"` では、dataに定義したtodos配列内のオブジェクトを一つずつ取り出し、todoに入れる、という処理をしています。
また、`v-for` ディレクティブを記載したhtml要素をtodoの分だけ繰り返します。

[リストレンダリング](https://jp.vuejs.org/v2/guide/list.html)

取り出したtodoの要素へのアクセスは`todo.text, todo.done` のようにアクセスできます。
`{{ todo.text }}`とすることで`<template>` のからもアクセスできます。
ここでは各todoには、text（todoの内容）とdone（todo済かどうかのフラグ）を定義しています。

## Todoの追加機能

これでtodoの一覧表示が出来たので、次にtodoの追加機能を作ります。

todoリストにtodoを追加していくには、v-forで表示しているtodos配列に要素を追加していけば良さそうです。
また、追加する内容は画面のテキストボックスの入力値を使用すれば良さそうですね。

### 双方向バインディング

従来であれば、clickイベントか、enterイベントの監視して、inputの中身を取得、…のようにすると思いますが、ここではVueの双方向バインディングを使ってみます。
双方向バインディングを使うと、js側で値を変更すれば画面側に反映され、画面側で値を変更すればjs側に反映されます。
Vueコンポーネント側でnewTodoというデータを追加し、`<input>` タグにバインディングしてみましょう。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/cc50c588d015be8ac2beaa89f4e2bb07bed8ead0

```html:src/components/Hello.vue
<template>
  <div>
    {{ msg }}
    <form>
      <button>ADD TASK</button>
      <button>DELETE FINISHED TASKS</button>
      <p>input: <input type="text" v-model="newTodo"></p>
      <p>task: {{ newTodo }}</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"
             v-for="todo in todos">
        <input type="checkbox"><button>EDIT</button>{{ todo.text }}
      </label>
    </div>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data: function() {
    return {
      msg: 'Welcome to Your Vue.js App',
      todos : [
        {text : 'vue-router', done: false},
        {text : 'vuex', done: false},
        {text : 'vue-loader', done: false},
        {text : 'awesome-vue', done: true },
      ],
      newTodo: ""
    }
  }
}
</script>

<style lang="scss" scoped>
/*省略*/
</style>
```

上手く行けば下のように、入力した値と連動してnewTodoが更新されるのが分かると思います。

![双方向バインディング](https://qiita-image-store.s3.amazonaws.com/0/67790/c0e24c9b-a8cf-618e-670c-8e6ce5061445.png)

あとはclickイベントかenterイベントに紐付けてnewTodoをtodosに追加してあげれば、todoの追加機能はできそうですね。

### v-onディレクティブ

vueにはイベントハンドリングのディレクティブがあるので、それを利用してADD TASKボタンが押されたらnewTodoをtodosに追加という処理を加えます。
（今更ですが、TodoとTaskという単語が混在していてよくないですね・・）

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/06b522cdbbeeaad51bf99fe638ceebca64ba7503

```html:src/components/Hello.vue
<template>
  <div>
    {{ msg }}
    <form>
      <button v-on:click="addTodo()">ADD TASK</button>
      <button>DELETE FINISHED TASKS</button>
      <p>input: <input type="text" v-model="newTodo"></p>
      <p>task: {{ newTodo }}</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"
             v-for="todo in todos">
        <input type="checkbox"><button>EDIT</button>{{ todo.text }}
      </label>
    </div>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data: function() {
    return {
      msg: 'Welcome to Your Vue.js App',
      todos : [
        {text : 'vue-router', done: false},
        {text : 'vuex', done: false},
        {text : 'vue-loader', done: false},
        {text : 'awesome-vue', done: true},
      ],
      newTodo: ""
    }
  },
  methods: {
    addTodo: function(event) {
      let text = this.newTodo && this.newTodo.trim()
      if (!text) {
        return
      }
      this.todos.push({
        text: text,
        done: false
      })
      this.newTodo = ''
    },
  }
}
</script>

<style lang="scss" scoped>
/*省略*/
</style>
```

`v-on:click="addTodo()"` がイベントハンドリングをしている部分です。`v-on` がディレクティブ、`:click` で何のイベントを監視するか、`="addTodo()"` に内容を記載します。
また、addTodo()はVueコンポーネントのmethodsオプションに記載します。ここではnewTodoに何か入っていれば、todosに追加し、newTodoを空にする、という処理をしています。
コンポーネント内のdataにアクセスする時は`this` で参照します。
また`v-on` ディレクティブは`@click="method"`のように省略記法があります。

[イベントハンドリング](https://jp.vuejs.org/v2/guide/events.html)

## Todoの削除機能の追加

これで、todoリストへの追加機能が出来ました。
次に、終了したtodoの削除機能を追加してみます。
先程、todoにはdoneというbooleanを追加しているので、これもnewTodoと同様に、リストレンダリングしたcheckboxにバインディングします。
また、DELETE FINISHED TASKSが押下されたら`todo.done===true` のtodoを削除してあげます。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/03619d921d285683527cf64da408541ffb97756a
（keyup.enterイベントを削除しているdiffも出ますが気にせず、、）

```html:src/components/Hello.vue
<template>
  <div>
    <form>
      <button @click="addTodo()">ADD TASK</button>
      <button @click="removeTodo()">DELETE FINISHED TASKS</button>
      <p>input: <input type="text" v-model="newTodo"></p>
      <p>task: {{ newTodo }}</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"
             v-for="todo in todos">
        <input type="checkbox" v-model="todo.done"><button>EDIT</button>{{ todo.text }}
      </label>
    </div>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data: function () {
    return {
      msg: 'Welcome to Your Vue.js App',
      todos : [
        {text : 'vue-router', done: false},
        {text : 'vuex', done: false},
        {text : 'vue-loader', done: false},
        {text : 'awesome-vue', done: true},
      ],
      newTodo: ""
    }
  },
  methods: {
    addTodo: function(event) {
      let text = this.newTodo && this.newTodo.trim()
      if (!text) {
        return
      }
      this.todos.push({
        text: text,
        done: false
      })
      this.newTodo = ''
    },
    removeTodo: function (event) {
      for (let i = this.todos.length - 1; i >= 0; i--) {
        if (this.todos[i].done) this.todos.splice(i, 1)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
/*省略*/
</style>
```

これで、画面のcheckboxの変化と連動して、todo.doneのtrue/falseが切り替わるようになりました。
また、removeTodoでは、todosを走査し、todo.doneがtrueであれば配列から削除しています。
このとき、todosに対し破壊的な操作をすることから、配列の長さは動的に変わります。
そのため配列はtodos.lengthから0へ向かって走査されていることに注意して下さい。

## Todoの編集機能の追加

これで一括削除機能が追加できました。
あとはtodoの編集機能ができれば一先ず完成です。
当初、EDITボタンを押下 → 編集画面ダイアログが表示 のように編集することを想定していましたが、ここも双方向バインディングと、v-ifディレクティブを使うことで簡単に実装してしまいます。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/38cf6a941c74708080befe03b48618af7a0d9100

```html:src/components/Hello.vue
<template>
  <div>
    <form>
      <button @click="addTodo()">ADD TASK</button>
      <button @click="removeTodo()">DELETE FINISHED TASKS</button>
      <p>input: <input type="text" v-model="newTodo"></p>
      <p>task: {{ newTodo }}</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"
             v-for="todo in todos">
        <input type="checkbox" v-model="todo.done">
        <input type="checkbox" v-model="todo.editing">
        <input v-if="todo.editing" v-model="todo.text" @keyup.enter="todo.editing = !todo.editing">
        <span v-else>{{ todo.text }}</span>
      </label>
    </div>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data: function () {
    return {
      msg: 'Welcome to Your Vue.js App',
      todos : [
        {text : 'vue-router', done: false, editing: false},
        {text : 'vuex', done: false, editing: false},
        {text : 'vue-loader', done: false, editing: false},
        {text : 'awesome-vue', done: true, editing: false},
      ],
      newTodo: ""
    }
  },
  methods: {
    addTodo: function(event) {
      let text = this.newTodo && this.newTodo.trim()
      if (!text) {
        return
      }
      this.todos.push({
        text: text,
        done: false,
        editing: false
      })
      this.newTodo = ''
    },
    removeTodo: function (event) {
      for (let i = this.todos.length - 1; i >= 0; i--) {
        if (this.todos[i].done) this.todos.splice(i, 1)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
/*省略*/
</style>
```

まずはtodoにeditingを追加しました。このフラグを見て編集している／していないを切り替えることにします。
加えて、EDITボタンはtodo.editingとバインディングしたチェックボックスに変更しました。

`v-if` ディレクティブを使用することで、要素の表示／非表示を切り替えることができます。
ここではtodo.editingを参照して、

* trueだったら、todo.textをバインディングし、keyup.enterイベントでtodo.editingを反転させる、`<input>` タグ
* falseだったら、todo.textをそのまま出力する`<span>` タグ

を`v-if`, `v-else` にそれぞれ追加しました。

editingにバインディングしたチェックボックスを切り替えることで、素のtodo.text／todo.textの入った`<input>` タグ、と切り替わることが確認できたでしょうか？
最終的に以下のような画面になります。

![完成形](https://qiita-image-store.s3.amazonaws.com/0/67790/7c12a5a1-70cb-5601-3525-ef4d28863b29.png)

これで、

1. Todoはリストで一覧表示すること
* Todoはテキストボックスから追加できること
* それぞれのTodoにはチェックボックスが付いており、それを切り替えることでTodoの状態（未達成／達成済）を切り替えること
* チェック済のTodoを一括で消すボタンがあること
* それぞれのTodoは編集可能なこと

を満たすTodoリストが完成しました。ここまできたら、あとは装飾ですね。

## v-bindディレクティブでclass制御

チェック済の項目については薄い青色で表示するようにしてみます。
`v-if` ディレクティブを使って、todo.doneを見て、文字色青色のcssを付与したタグを出力／素のタグを出力…のようにDOMの描画で分けることも可能ですが、`v-bind` ディレクティブを使って、classを付与することで切り替えてみましょう。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/2fbb3cd80cca9ca4bd1abd8595fece173828f4db

```html:src/components/Hello.vue
<template>
  <div>
    <form>
      <button @click="addTodo()">ADD TASK</button>
      <button @click="removeTodo()">DELETE FINISHED TASKS</button>
      <p>input: <input type="text" v-model="newTodo"></p>
      <p>task: {{ newTodo }}</p>
    </form>
    <div class="task-list">
      <label class="task-list__item"
             v-for="todo in todos"
             v-bind:class="{ 'task-list__item--checked': todo.done }">
        <input type="checkbox" v-model="todo.done">
        <input type="checkbox" v-model="todo.editing">
        <input v-if="todo.editing" v-model="todo.text" @keyup.enter="todo.editing = !todo.editing">
        <span v-else>{{ todo.text }}</span>
      </label>
    </div>
  </div>
</template>

// 省略
```

html要素に対するclassのバインディングには、

* オブジェクト構文
* 配列構文
* 
の書き方がありますが、ここではオブジェクト構文で書いています。

[クラスとスタイルのバインディング](https://jp.vuejs.org/v2/guide/class-and-style.html)

todo.doneがtrueと評価される場合、class='task-list__item--checked'が付与されます。
また、v-bind:classはプレーンなclass属性がある要素に書いても大丈夫です。


これで装飾も完了しました。
ここまで出来たらアプリケーションを配信してみましょう。

## GitHub-Pagesを使って配信する

アプリケーションの配信にはGitHub-pagesを使います。
これはGitHubのリポジトリに対応して、静的ファイルをホスティングできる仕組みです。
プロダクトのランディングページなどにも使用されます。

まずは配信用の静的ファイルをビルドしてみましょう。この仕組みもvue-cliで用意されています。

```console
% npm run build
```

デフォルトの設定だと./distが配信用ディレクトリとして作成されるはずです。
個人で配信環境を持っている人はこれでOKですが、今回はGitHub-pagesでホストするので、少しだけ設定を変えます。

GitHub-pagesではリポジトリルート直下の./docsディレクトリが配信されるので（ここは設定によります）、./docsディレクトリを生成するように変更します。

diff: https://GitHub.com/sin-tanaka/vuejs_tutorial_todo_management/commit/b6fb359da3dc7080682cf703613a2328c9679a95

```js:config/index.js
// see http://vuejs-templates.GitHub.io/webpack for documentation.
var path = require('path')

module.exports = {
  build: {
    env: require('./prod.env'),
    index: path.resolve(__dirname, '../docs/index.html'),
    assetsRoot: path.resolve(__dirname, '../docs'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './',
    productionSourceMap: true,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
// 省略
}
```

設定後、再度`npm run build` するとdocsディレクトリが作成されるので、プロジェクトをGitHubにpushします（gitの設定は割愛）。
リポジトリのSettingから、./docsをGitHub-pagesとして配信するように設定します。

![05.png](https://qiita-image-store.s3.amazonaws.com/0/67790/fce48b75-75cb-04bb-5075-0e8b2db8598d.png)

これで、`https://<usename>.github.io/<repository_name>/`にアクセスすると、./docsが配信できていることを確認できるかと思います。

以上でチュートリアルは終了です。

## まとめ

いかがだったでしょうか？今回はvue-cliを使って、Vue.jsの機能を活用したTodoリストを作成しました。
Vue.jsやvue-cliの便利さが体感できたでしょうか。jQueryなどと比べてもかなり楽に作成できたことかと思います。

今回使用した双方向バインディングなどは、vue-cliを使わずとも、CDNで配信されているVue.jsを読み込むことでも既存環境に簡単に組み込むことが可能です。

ここまでで基本的なことは一通り学べたかと思います。あとは、

* インスタンスのオプション（computed、ready、created、watch）
* ルーティング(vue-router)
* コンポーネント分割
* axiosを使ったリクエスト送信
* VueのFluxアーキテクチャ実装Vuex

などを学ぶことでより理解が深まるかと思います。
ご指摘・ご要望あれば、ご一報下さい！
