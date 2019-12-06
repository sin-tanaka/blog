---
title: 'Slackで動的なユーザグループを作成しようとして失敗した'
description: 'Slackで動的なユーザグループを作成しようとして失敗した記録です'
date: 2019-12-06T01:56:12.789Z
thumbnail: /images/uploads/lifecycle.png
categories:
  - Python
tags:
  - Python
---

この記事は https://qiita.com/sin_tanaka/items/77f7b6bf2c11d41617fc にも紐付いています。

## TL;DR

* Slackで動的なユーザグループがあると便利そう
  * `@here` みたいな
* Slackの `usergroups.users.update` APIを使うと行けるのでは？
* `Chalice` (Python)と `AWS Lambda` 使って定期実行したらできた
* できたけど問題が発生したので辞めた
* 用途にあってるのか技術検証大事


## 前置き

弊社は歴史が古いながらも、「コアタイムなしのフルフレックス&リモートワーク推奨」な自由な働き方を売りにしている会社です。
出社時間はみんなバラバラですし、3割ぐらいはリモートワーカーです。よって、オフィスを構えているものの全員が出社することは稀です。

しかし働き方が変わると、**議論すべき良し悪しや、解決すべき課題**も発生します。
本記事はリモートワークの課題を解決しようとした話です。

## リモートワーク時のチャット運用の課題

昨今、 `@here` , `@channel` 禁止等、各社のSlackの運用の話題になるのを見かけます。
通知は便利ですがプログラマの集中力を乱す原因にもなるからです。

そんな中、弊社Slackでこんな `@here` メンションがありました(自分のポストです、、)

![スクリーンショット 2019-12-05 18.48.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/67790/f9286cd9-581b-7eea-3713-a132665605ea.png)

どうでしょうか？状況としては、「少し急ぎ」「アクティブなユーザにのみ通知したい」な状況なので、一般的な `@here` の使い方かと思います。

しかしリモートワーカーからしたらどうでしょうか
リモートワーカーにとって社屋の会議室、応接室の使用状況は共有されなくても問題ないはずです。
 
つまりリモートワーカーにとってこの通知は不要なのです。

もちろんこれは通知をされる側だけでなく `@here` で通知する側もリモートワークの人に通知飛ばすの申し訳ないな、、と感じるかもしれません。

「リモートワーク推奨・・・だけど関係ない通知たくさんするよ！！」な会社だったらプログラマにとって働きやすい環境とは言えないのでは…？

このように、リモートワーカーが増えるとチャットの **通知先を設計** することが重要になってきます。

## 動的なメンション

そこで考えたのが**動的なメンション**です。
ある条件を満たすユーザのみに通知されるメンションがあればこの問題を解決できると考えました。

今回の例でいうと、**「今日出社している人のみに通知されるメンション」**となります。

そういえば、 `@here` もチャンネル内のアクティブなユーザのみに通知する、という意味で動的なメンションですね。

## 実装の検討

弊社には内製の行き先ボードと、そのREST APIがあります。
これを利用すればオフィスに出社している人は一覧で取れる。
なのであとはなんとかオフィスに出社している人一覧に向けてSlack上でメンションできればよさそうです。

Slack側にいい感じのAPIがないか調べていると `usergroups.users.update` というAPIを発見しました。

[usergroups\.users\.update method \| Slack](https://api.slack.com/methods/usergroups.users.update)

usergroupにグループのID、usersにユーザのIDの配列を渡すことでユーザーグループのユーザーを更新できるというものです。

なので、

1. 行き先ボードのAPIからオフィスに出社している人一覧を取得
2. 一覧をSlackのユーザIDに変換
3. 予め作っておいたユーザグループに対して `usergroups.users.update` を実行してユーザーを更新する
4. これを定期実行する

を実現できればいけそうです！

## 利用技術

* 定期実行できる
* 誰でもメンテできる
* できれば運用コストをかけない

を考え、 `AWS Lambda` を使用することにしました。

Lambdaであれば仮に平日(20日)の定時 + 前後1時間(10時間)の間毎分の感覚で動作しても、
20 * 10 * 60 = 12,000
なので毎月1,000,000リクエストの無料枠には余裕で収まります、

また、Lambdaのデプロイには `chalice` を使いました

[aws/chalice: Python Serverless Microframework for AWS](https://github.com/aws/chalice)

`chalice` は `Flask` ライクのデコレータベースでLambdaを記述でき、周辺サービスの作成(削除)までやってくれるフレームワークです。
Lambdaは、Http(s)を受け取りたい場合、 `APIGateway` との連携が必要だったりして面倒ですが、その辺の面倒をみてくれます。

例えば、今回の要件である定期実行も `chalice` の構文で書くことが可能です。

```python:5分毎に定期実行する例
from chalice import Chalice, Rate

app = Chalice(app_name="helloworld")

# Automatically runs every 5 minutes
@app.schedule(Rate(5, unit=Rate.MINUTES))
def periodic_task(event):
    return {"hello": "world"}
```

最終的なコードは以下のようになりました（自社APIの利用部分は割愛）

```python:src/update_usergroup.py
@app.schedule(Cron('0', '23-9', '?', '*', 'SUN-THU', '*'))
def update_in_office_usergroup(event):
    OAUTH_TOKEN = 'token_xxxx_xxxx'
    # 事前に作成たユーザグループのUID
    in_office_usergroup_uid = 'some_usergorup_uid'

    # 〜省略〜

    # APIから取得したオフィスに出社しているユーザのSlackID一覧
    office_user_slack_ids = ['some', 'user', 'uids']

    try:
        data = urllib.parse.urlencode(
            (
                ("token", OAUTH_TOKEN),
                ("usergroup", in_office_usergroup_uid),
                ("users", ','.join(office_user_slack_ids)),

            )
        )
        data = data.encode("ascii")
        request = urllib.request.Request(
            "https://slack.com/api/usergroups.users.update",
            data=data,
            method="POST"
        )
        request.add_header(
            "Content-Type",
            "application/x-www-form-urlencoded"
        )

        with urllib.request.urlopen(request) as r:
            resp = json.loads(r.read())
        logger.info(resp)
        return {'ok': True}
    except Exception as e:
        logger.error(e)
        return {'ok': False}
```

Lambdaを使うので外部パッケージは入れずに `urllib` を使用してます。

またUTCの調整でcron式を9時間ずらして、日曜-木曜の23-9時としています(日本時間で月曜-金曜の9-18時になる)。
(これってなんかいい方法ないんでしょうか。。)

## 完成したが・・・

`@in_office` と名付けてドヤ顔でリリースしました。

![スクリーンショット 2019-12-06 10.18.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/67790/9edbbe5a-1e14-324e-9445-ed0e53cc1816.png)

しかしこんな通知が頻繁にくるという報告が。。

![スクリーンショット 2019-12-06 10.20.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/67790/e6337c24-bdf7-eeab-9d30-2af1f0f54913.png)

どうやらユーザグループの追加/削除するとSlackbotから通知が飛んでしまうようです。
無駄な通知を減らす目的で作ったのに、逆に通知が増えてしまいました。これでは使い物にならない・・・

「API経由であれば通知を切るようなオプションがあるかも？」と思い、調べてみましたがどうやら無いようでした。

ここで手詰まりとなり、せっかく作った機能ですが運用を停止しました。元の木阿弥です。

## 振り返り

技術検証が足りてなかったのが一番の原因です。

「APIがある！アイデアもある！いけそう！」となった時点で走りはじめて、小さなコードで試すこともしませんでした。
選定した技術で問題を解決できるのか？問題解決に合った手段なのか？の検討は大事ですね。

一方で、チャットの通知を設計する発想自体は良かった気がしてます。

まぁしょせん個人開発なので失敗したことも釣果ではありますし、これも技術検証のうちでしょう！

動的なメンションかなりいいアイデアだと思うんだけどなぁ。。
この記事を読んだどなたか、実現したらぜひ教えて下さい。

## おまけ

拙い英語で公式に問い合わせたところ丁寧な回答がありました。
現状(問い合わせ時点の2019年10月)のusergroupAPIでは通知を切ることはできないがフィードバックチームに伝えて将来的にはできるようにしたいとのこと。

実装されたらまたチャレンジしてみたいと思います！

