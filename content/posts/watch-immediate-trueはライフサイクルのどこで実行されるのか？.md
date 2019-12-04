---
title: 'watch immediate: trueはライフサイクルのどこで実行されるのか？'
description: ' immediateオプション付きのwatchはライフサイクルにおけるどのタイミングで呼ばれるのか？ という疑問が湧いたので調べてみました'
date: 2019-12-04T01:56:12.789Z
thumbnail: /images/uploads/lifecycle.png
categories:
  - Vuejs
tags:
  - Vuejs
---
この記事は https://qiita.com/sin_tanaka/items/64b4a48bcb6dac924380 にも紐付いています。

## 前置き

Vue.jsにはVueインスタンス(コンポーネント)上のデータの変更を監視する [watch](https://jp.vuejs.org/v2/api/#watch) というプロパティがあります。

さらに、 `watch` には `immediate` というオプションがあります。

`watch` は通常監視を始めて、データが変わった直後にコールバックが呼ばれますが、 `immediate` オプションを付与した `watch` は監視を始めた直後に一回コールバックが呼ばれます。

また、Vueにはインスタンスのライフサイクルに合わせて関数を実行する `ライフサイクルフック` という仕組みがあります。

そこで、 **immediateオプション付きのwatchはライフサイクルにおけるどのタイミングで呼ばれるのか？** という疑問が湧いたので調べてみました。

![ライフサイクルフック](https://jp.vuejs.org/images/lifecycle.png)

画像は [Vue インスタンス — Vue\.js](https://jp.vuejs.org/v2/guide/instance.html#%E3%82%A4%E3%83%B3%E3%82%B9%E3%82%BF%E3%83%B3%E3%82%B9%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB%E3%83%95%E3%83%83%E3%82%AF) より引用

## TL;DR

* watchのコードリーティングしてみた
* **実行タイミングは `beforeCreate` と `created` のあいだ** 
  * ドキュメントには記載無さそう?現状のコードではこのタイミングってぐらいなはず

## ひとまず実行してみる

以下のコードで試してみたところ、`immediate` 付きの `watch` は `beforeCreate` と `created` の間に実行されました。

https://codepen.io/sin-tanaka/pen/ExaxrZx

```js
new Vue({
  el: '#app',
  data: function() {
    return {
      helloWorld: 'HelloWorld'
    };
  },
  beforeCreate: function() {
    console.log('call beforeCreate');
  },
  created: function() {
    console.log('call created');
  },
  mounted: function() {
    console.log('call mounted');
  },
  watch: {
    helloWorld: {
      handler: v => console.log('call watch', v),
      immediate: true
    }
  }
})
```

```
call beforeCreate
call watch HelloWorld
call created
call mounted
```

ライフサイクルの図で言うところの `Init Injections & Reactivity` でWatchを仕掛けているようです。

## Vuejsのコードを読んでみる

改めてwatchのドキュメントを読んでみましたが、上記の動作を保証するような文言はなさそうでした。

そこで、2019.12.3時点でのdevブランチのコードを読んでみることにします

[vuejs/vue: 🖖 Vue\.js is a progressive, incrementally\-adoptable JavaScript framework for building UI on the web\.](https://github.com/vuejs/vue)

## new Vueは何をしているのか？

`new Vue` したときにどのようなコードを実行しているのかを追ってみます
`package.json` の `scripts` → `rollup` → `runtime` →…のように追っていくと `new Vue` の実態は以下のようでした。

https://github.com/vuejs/vue/blob/dev/src/core/instance/index.js

```js:src/core/instance/index.js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

`function Vue` new演算子付きで呼ばれたときのみ、 `this._init(option)` を実行しています。実質コンストラクタですね。

`_init` は Vueの中に定義されていないので、 `initMixin` `stateMixin` `eventsMixin` `lifecycleMixin` `renderMixin` 辺りで `Vue.prototype._init` を仕掛けているとみます。


### initMixin

`initMixin` を読んでみると `Vue.prototype._init = function (options?: Object)` という記述がありました。この関数でコンストラクタに該当するコードを仕込んでいるようです。

```js:src/core/instance/init.js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    /**
    * 中略
    */
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    /**
    * 中略
    */
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

また、 `callHook(vm, 'beforeCreate')` と `callHook(vm, 'created')` の関数呼び出しの行があります。これは名前の通り、ライフサイクルフックを実行している関数でした。

その間には `initInjections(vm)` `initState(vm)` `initProvide(vm)` という関数呼び出しがありました。
これはまさしくライフサイクルの図でいうところの **`Init Injections & Reactivity`** に該当する関数に見えます。

### initState

`initState` 関数を見てみると `initWatch` 関数を実行している行がありました。

```js:src/core/instance/state.js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

さらに `initWatch` 関数を追ってみます


```js:src/core/instance/state.js
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```

まだ `immediate` の記述はナシ `createWatcher` を追います

```js:src/core/instance/state.js
function createWatcher (
  vm: Component,
  expOrFn: string | Function,
  handler: any,
  options?: Object
) {
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  return vm.$watch(expOrFn, handler, options)
}
```

`vm.$watch` を実行しているようなので、これも `_init` のように `prototype` に仕掛けている箇所を探してみます

## vm.$watchはどこで仕掛けているのか？

`index.js` で実行している `stateMixin` で仕掛けているようでした

```js:src/core/instance/state.js
export function stateMixin (Vue: Class<Component>) {
  /**
  * 中略
  */
  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      try {
        cb.call(vm, watcher.value)
      } catch (error) {
        handleError(error, vm, `callback for immediate watcher "${watcher.expression}"`)
      }
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
}
```

ここで `watch` のオプション `immediate` がtruthyであるとき `cb.call(vm, watcher.value)` を実行しているのが分かります。長かった・・・

よって、コードベースでも見ても **`immediate: true` の `watch` の実行タイミングは `beforeCreate` と `created` の間** であることが分かりました

## おまけ

コードリーティングしてたら `$watch` が何やら `unwatchFn` なるfunctionを返しているのを発見しました。
名前のとおりですが、 `$watch` の戻り値をコールすると監視が解除されます。

https://codepen.io/sin-tanaka/pen/oNgNmGz
