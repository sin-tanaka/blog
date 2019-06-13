---
title: ' Chalice(Lambda + API Gateway)を使ってInteractiveComponentを扱うSlackAppを作る'
description: ' Chalice(Lambda + API Gateway)を使ってInteractiveComponentを扱うSlackAppを作る'
date: 2019-05-23T08:39:57.020Z
thumbnail: /images/uploads/images.png
categories:
  - slack
  - chalice
---
## SlackのInteractiveComponent

[Making messages interactive \| Slack](https://api.slack.com/interactive-messages)

SlackBotでもInteractiveComponent自体を投稿することはできるが(単にattachmentsを付与すれば良い)、アクションのリダイレクト先を指定することができないのでSlackAppを作成する必要がある

※: SlackBotとSlackAppは別物

## 想定仕様

- EventAPIとInteractiveComponentのリダイレクト先に指定するサーバはAPI Gateway + Lambdaを使う
- Flaskライクに書いたAPIをAPI Gateway + Lambda etc..へデプロイできるフレームワークChaliceを使う
- SlackAppにメンションを飛ばすとInteractiveComponentを返す
    - EventSubscribe(EventAPI) で `app_mention` イベントをlistenする
    - RedirectURLは `foo-bar-domain/api` を想定
- InteractiveComponentにはYes/Noボタンを付ける
    - どちらかのボタンが押されたとき対応する文章を返す
    - RedirectURLは `foo-bar-domain/api/reaction` とする


## SlackAppの作成

[Slack API \| Slack](https://api.slack.com/) でStartBuildingボタンを押すとアプリを作成することができます

つづけて、
- BotUserの作成
- InteractiveComponentsの有効化
- Permissionの設定
    - chat:write:bot
    - bot BotUser作成したときに付与される

をしておきます。最後にInstallAppすることでWorkspaceへAppをインストールします

RedirectURLはデプロイ後、URLが決定したら設定します

## ChaliceでEventAPI、InteractiveComponentのRedirectを受けるAPIを作る

chaliceをインストールしてプロジェクトを作成します

[AWS Chalice — Python Serverless Microframework for AWS 1\.8\.0 documentation](https://chalice.readthedocs.io/en/latest/)

```sh
$ pip install chalice
$ chalice new-project foo_bar_api
```

AWSのCredentialsが正しくlocalで設定されてる場合この時点でデプロイ可能です

IAM、APIGateway、Lambdaが作成され、APIGatewayのURLが表示されます

```sh
$ chalice deploy
```

削除も簡単です

紐づくIAMまで削除してくれます

```sh
$ chalice delete
```

localserverで開発も可能です

hotreload付き

```sh
$ chalice local
```

ひとまずlocalで開発したい、ログが見たい場合は `chalice local`  + `ngrok` が良さそうな雰囲気です

[ngrok \- secure introspectable tunnels to localhost](https://ngrok.com/)


app.pyを編集します

```python
import json
import logging
import urllib

from chalice import Chalice

app = Chalice(app_name='foo-bar')

# ログレベル: =INFOの情報も出力する
app.debug = True
logger = logging.getLogger()
logger.setLevel(logging.INFO)


# SlackAppのSettings > Basic InformationのApp Credentials参照
BOT_OAUTH_TOKEN = "xoxb-foo-bar-your-credentials"
# http requestのtokenフィールドの値
VERIFICATION_TOKEN = "foo-bar-your-verificationtoken"

# InteractiveComponentの実態 attachmentsに載せる情報
attachments_json = [
    {
        "fallback": "Upgrade your Slack client to use messages like these.",
        "color": "#258ab5",
        "attachment_type": "default",
        "callback_id": "hogehoge",
        "actions": [
            {
                "name": "yes",
                "text": "YES",
                "value": "y",
                "type": "button"
            },
            {
                "name": "yes",
                "text": "NO",
                "value": "n",
                "type": "button"
            }
        ]
    }
]


@app.route('/', methods=['POST'])
def handler():
    request = app.current_request
    # EventAPIのRedirectURLの認証のためchallengeフィールドの値をそのまま返す
    if "challenge" in request.json_body:
        return request.json_body["challenge"]

    slack_event = request.json_body['event']

    # ループしないための対策 e.g. appがappへのmentionを含むケース
    if "bot_id" in slack_event:
        logger.warning("Ignore integration bot message")
    else:
        response = 'which is your choice'
        respond(response, slack_event, attachments_json)

    return "200 OK"


# InteractiveComponentのデータはx-www-form-urlencodedで渡される
@app.route('/reaction', methods=['POST'], content_types=["application/x-www-form-urlencoded"])
def reaction_handler():
    request = app.current_request
    body = urllib.parse.unquote_to_bytes(request.raw_body).decode()
    payload = body.split('=', 1)[1]
    payload_dict = json.loads(payload)

    # 選択された値を取得
    choose_value = payload_dict["actions"][0]["value"]
    if choose_value == 'y':
        response = "your choice is yes "
    else:
        response = "your choice is no"
    # InteractiveComponentはreturnしたstrで置き換えられる
    # 連打防止の為 InteractiveComponentは残らない設計がbetterとされている
    return response


def respond(response, slack_event, attachments=None):
    channel_id = slack_event["channel"]
    data = urllib.parse.urlencode(
        (
            ("token", BOT_OAUTH_TOKEN),
            ("channel", channel_id),
            ("text", response),
            ("attachments", attachments)
        )
    )
    data = data.encode("ascii")

    # chat.postMessageへPOSTする
    request = urllib.request.Request(
        "https://slack.com/api/chat.postMessage",
        data=data,
        method="POST"
    )
    request.add_header(
        "Content-Type",
        "application/x-www-form-urlencoded"
    )
    urllib.request.urlopen(request).read()

```

デプロイします

```sh
$ chalice deploy
```

- /api
- /api/reaction

のURLが表示されると思います

それぞれ、EventSubscribe, InteractiveComponent のRedirectURLに設定します


## 試してみる

Slackでアプリのbotユーザに対し、メンションを飛ばすことで機能を確認できるかと思います

1つのアプリで複数のInteractiveComponentに対応する場合、メンションしたときのtextや、attachmentのcallback_idで分別することになると思います
