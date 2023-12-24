---
title: Vuetify 2.0へのアップグレード時にやったこと
tags:
  - Vue.js
  - Vuetify
  - vuetifyjs
private: false
updated_at: '2019-08-21T19:49:13+09:00'
id: 3bfedbe58c71ae2c8cd4
organization_url_name: fujitsu
slide: false
ignorePublish: false
---
# 概要
2019/7/23に Vuetify 2.0が正式リリースされました。
旧バージョン(1.5.5)からアップグレードする際に少し詰まったので、やったことを備忘録としてまとめておきます。

# やったこと

### package.json
バージョンアップするためにpackage.jsonを修正。
修正後に npm install します。

```diff:package.json抜粋
"dependencies": {
-    "vuetify": "^1.5.5"
+    "vuetify": "^2.0.0"
},
"devDependencies": {
-    "@vue/cli-plugin-babel": "^3.9.0",
-    "@vue/cli-plugin-eslint": "^3.9.0",
-    "@vue/cli-service": "^3.9.0",
+    "@vue/cli-plugin-babel": "^3.10.0",
+    "@vue/cli-plugin-eslint": "^3.10.0",
+    "@vue/cli-service": "^3.10.0",
-    "stylus": "^0.54.5",
-    "stylus-loader": "^3.0.1",
+    "sass": "^1.17.4",
+    "sass-loader": "^7.1.0",
-    "vuetify-loader": "^1.0.5"
+    "vuetify-loader": "^1.2.2"
}
``` 

### main.js
Vueインスタンスの作成時に、Vuetifyのインスタンスをオプションで渡すようにしました。

```diff:main.js
import Vue from 'vue'
import 'material-icons/iconfont/material-icons.css'
-import './plugins/vuetify'
+import vuetify from './plugins/vuetify'
import App from './App.vue'
import router from './router'

Vue.config.productionTip = false

new Vue({
  router,
+ vuetify,
  render: h => h(App)
}).$mount('#app')
```

### vuetify.js
Vuetifyのインスタンスをexportするようにしました。
これをmain.jsでimportします。

```diff:vuetify.js
import Vue from 'vue'
import Vuetify from 'vuetify/lib'
-import 'vuetify/src/stylus/app.styl'
+import 'vuetify/src/styles/main.sass'
-
-Vue.use(Vuetify, {
-  iconfont: 'md',
-}) 
+
+Vue.use(Vuetify);
+export default new Vuetify({
+  icons: {
+    iconfont: 'mdi',
+  },
+});
```

### App.vue
\<v-app>で全体を囲うようにします。
これがないと、配下の全要素でcolor属性が反映されなくなりましたので注意。

```diff:App.vue
<template>
+  <v-app>
    ~~~
+  </v-app>
</template>
```

### その他vueファイル

コンポ名や属性名が変わった部分を修正しました。
私の場合では、下記を修正。

|修正前                 |修正後                |
|:--------------------:|:--------------------:|
|\<v-list-tile>        |\<v-list-item>        |
|\<v-list-tile-content>|\<v-list-item-content>|
|\<v-list-tile-action> |\<v-list-item-action> |
|\<v-btn flat>         |\<v-btn text>         |

動かなくなったコンポがあれば、下記ドキュメントページでコンポ名の変更等がないかを調べてみましょう。
[v1.5のドキュメント](https://v15.vuetifyjs.com/ja/components/api-explorer)
[最新のドキュメント](https://vuetifyjs.com/ja/components/api-explorer)

# おわりに
上記修正で、バージョンアップ後も問題なくアプリが動作するようになりました。
が、理由がよく分かっていないところも多いです。。
細かい仕様は追々調べていく予定。
