---
title: iOS12 で SMS で受け取った認証コードを補完候補に出すときは autocomplete 属性に one-time-code を指定する
description: |-
  iOS12ではセキュリティ周りのアップデートで、セキュリティコードの自動入力がサポートされました。
  主な機能として、サインイン時のパスワード自動入力や、アカウント作成時の自動パスワード生成などがサポートされました。
date: 2019-02-14T09:15:11.180Z
thumbnail: /images/uploads/ios-12-1200px.jpg
categories:
  - HTML
  - JavaScript
  - MobileSafari
  - iOS
---
iOS12ではセキュリティ周りのアップデートで、セキュリティコードの自動入力がサポートされました。

主な機能として、サインイン時のパスワード自動入力や、アカウント作成時の自動パスワード生成などがサポートされました。

以下のブログでネイティブアプリの実装を紹介しています。gif付きでわかりやすいです。

[すぐできる iOS Strong Password and Security Code AutoFill – PSYENCE:MEDIA](https://tech.recruit-mp.co.jp/mobile/post-17980)

そんなアップデートの中で、地味ではありますが、SMSで受けとった認証コードを補完候補に出す、という機能が追加されました（上記のブログ内のgif参照）。

本記事では、 `MobileSafari(WebView)` 内で有効にするのに手間取ったのでメモ。

# やったこと

記事のタイトル通りですが、 `<input>` タグの `autocomplete` 属性に `one-time-code` を指定することで実現可能です。

```html
<input id="single-factor-code-text-field" autocomplete="one-time-code"/>
```

[Enabling Password AutoFill on an HTML Input Element \| Apple Developer Documentation](https://developer.apple.com/documentation/security/password_autofill/enabling_password_autofill_on_an_html_input_element)

# まとめ

HTMLはどことなくエンジニアが軽視しがちな領域ですが、正しく構造化したり、適切な属性を付与することで、メンテナンス性やアクセシビリティの向上が見込めます。

地味ではありますが、少し手を加えるだけでユーザビリティ向上につながりますね。
