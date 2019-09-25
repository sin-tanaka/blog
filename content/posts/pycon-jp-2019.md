---
title: 'PyCon JP 2019で登壇 & 参加しました #pyconjp'
description: 昨年に続き今年もCfPが採択され登壇してきました
date: 2019-09-25T03:00:09.290Z
thumbnail: /images/uploads/8cfb736d-74c0-4c11-bcd2-f40efab44537_new.jpg
categories:
  - python
  - pyconjp
  - conference
---
ありがたいことに、昨年のPyCon JP 2018に続き、今年もCfPが採択されたので登壇 & 参加してきました。

- スライド: https://speakerdeck.com/sin_tanaka_21/pyconjp-2019
- 配信: https://youtu.be/lCQWLAJf6xQ

CfPや、当日の意識したこと等、メモ書き程度に残しておきます

---

## CfPについて

昨年は割と小賢しいタイトルをつけたなぁと自負していて、「実践入門」って一体なんだよって感じではあったので今年はシンプルに、「やりたいこと」と「PyCon自体のテーマ」をかけ合わせる感じにしました。

- PyCon JP 2018のスライド: https://speakerdeck.com/sin_tanaka_21/pyconjp-2018
- YouTube: https://youtu.be/zgTP_4-XEpw

決まったタイトルが `Pythonでライブをしよう!FoxDotを使った新時代のPython活用法` でした。

まぁ、「新時代」と謳いつつも、 `FoxDot` やライブコーディング自体は随分前から存在しているので微妙だなと思わんではないのですが、「PyConの場でライブコーディングの実演したら新時代感あるやろ！」と思って応募しました。

## 内容の検討

CfP出した段階で草案は考えていたのですが、内容を詰めたのは採択が決まってからです。

考えた結果、ライブコーディングのデモに全振りしつつ、自分が知識欲満たせるプレゼンが好きなので特殊メソッドのオーバーロードの解説をすることにしました。

あとは、当初メディア・アートとの関連も少し話す予定でしたが、

- 自分が詳しくないので付け焼き刃
- Pythonとの関わりが薄い

の理由で話さないことにしました。

## 当日のデモ

`FoxDot` だと奇数のハイハット作るのが楽だし、トラップ流行ってるしそれっぽいBPM/ビートでいくかーとか、電子音に合いそうだし小室っぽいコード進行にするかーとか、全部で5案ぐらい考えたんですが、最終的には、「変わったことはしないで、すごい風に魅せる」を念頭において、

- カノン進行
- 一般的な `House/Techno` 辺りのBPM
- `FoxDot` のbellが結構いい音 & カノン進行に合うので主軸
- メソッドチェーンで音を重ねる
- 転調で盛り上がりを作る
- 徐々に音数を増やす/減らす

なデモに仕上げました。

あとは、事前に用意したコードを徐々にコメントアウト解除する方式でやれば本番も大丈夫かな！

…と思ったのですが、むくむくと欲が出てきて、「本当のライブコーディングは事前に用意したコードをコメントアウトしておくのか…？」「その場でコーディングしてくのが本物のライブコーディング&カッコいいのでは…？」と思い立ち、PyCon前の土曜日にデモの案ができあがって、そこからはひたすら流れの暗記と練習をしました。

結果、本番でも無事成功し、やはりコメントアウト方式よりも説得力が出た気がしています。相当アドレナリン出ました。

後日同僚から「カンペあったの？」と聞かれたのですがカンペはないです。ライブ性のあるものに対し、カンペを要しないのはプレゼンとしてはバッドプラクティスかもですが、今回はギリギリまでデモを詰めていたためカンペを用意する余裕がなかったのと、「カンペを見ながらやる」というのも実はそこそこ練習が必要という考えのもと用意しませんでした。

## その他

今年からリアルタイムで配信されており、休憩時間にスマホで過去の配信を見ることができたので、かなり体験として良かったです。
今年は以下の辺りの発表を見ました。まだ全然観きれてない。。

- [メディアが運用すべき持続可能なVTuberをつくる技術\(加藤皓也/Hiroya Kato\) \- YouTube](https://www.youtube.com/watch?v=oXhGABVWV8k)
- [知ろう！使おう！HDF5ファイル！\(thinkAmi\) \- YouTube](https://www.youtube.com/watch?v=bSdRlfC2yqA)
- [基調講演\_Pythonで切り開く新しい農業\(小池 誠\) \- YouTube](https://www.youtube.com/watch?v=0bTPOsVvG7g)
- [Pythonで始めてみよう関数型プログラミング\(寺嶋 哲/Terajima Satoshi\) \- YouTube](https://www.youtube.com/watch?v=hGfWInLzKHQ)
- [Doujin\-activity with Ren'Py\(Daisuke Saito\) \- YouTube](https://www.youtube.com/watch?v=mOu6_6VVrRQ)
- [Djangoで実践ドメイン駆動設計による実装\(大島和輝\) \- YouTube](https://www.youtube.com/watch?v=9-OIHTJdDew)
- [Pythonと便利ガジェット、サービス、ツールを使ってセンシング〜見る化してみよう\(知野 雄二/Yuji Chino\) \- YouTube](https://www.youtube.com/watch?v=ZMjNAay-goQ&t=4s)
- [PythonとAutoML\(芝田 将\) \- YouTube](https://www.youtube.com/watch?v=Whkwu46DgBs&t=1614s)
- [PythonとGoogle Optimization Toolsの最適化ライブラリで、「人と人の相性を考慮したシフトスケジューラ」を作ってみた。\(鈴木庸氏\) \- YouTube](https://www.youtube.com/watch?v=xv_mFS4YmKE&t=1317s)

個人的には発表を受けて `Ren'py` について調べていたら近年の超名作 `Doki Doki Literature Club!` が `Ren'py` 製だったのが驚きでした(そういえば `Python` の `Exception` とか出てた)

[Steam：Doki Doki Literature Club\!](https://store.steampowered.com/app/698780/Doki_Doki_Literature_Club/)

大変おすすめです。

---

最後になりますが、発表内容の査読をしていただいたJSLの皆さん、当日発表を聞いていただいた皆さんありがとうございました。デモ終了時の大きな拍手は忘れられません。

そしてPyCon JPの運営スタッフの皆様、あれほどの規模のカンファレンスを自律性持ってやるのは並大抵のことではないと思います、本当にお疲れさまでした！
