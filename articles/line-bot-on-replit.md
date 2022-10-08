---
title: "LineBotをスマホとReplitで作ってみる"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linebot", "python", "replit"]
published: true
---

## 今回使用するもの

- Python
- [line-bot-sdk-python](https://github.com/line/line-bot-sdk-python)
- [Flask](https://github.com/pallets/flask)
- Replit(Pythonを動かすために)

## 準備

### Replit側

1. まずはReplitのアカウントを作成
@[card](https://replit.com/)

2. Pythonのreplを作成する
![](https://storage.googleapis.com/zenn-user-upload/d9b6d864f139-20221008.png)

3. main.pyにコードを貼り付ける

```py
import os
import sys
from argparse import ArgumentParser

from flask import Flask, request, abort
from linebot import (
    LineBotApi, WebhookHandler
)
from linebot.exceptions import (
    InvalidSignatureError
)
from linebot.models import (
    MessageEvent, TextMessage, TextSendMessage,
)

app = Flask(__name__)

# アクセストークの読み込み
channel_secret = os.getenv('LINE_CHANNEL_SECRET', None)
channel_access_token = os.getenv('LINE_CHANNEL_ACCESS_TOKEN', None)

line_bot_api = LineBotApi(channel_access_token)
handler = WebhookHandler(channel_secret)


@app.route("/callback", methods=['POST'])
def callback():
    signature = request.headers['X-Line-Signature']

    body = request.get_data(as_text=True)
    app.logger.info("Request body: " + body)

    try:
        handler.handle(body, signature)
    except InvalidSignatureError:
        abort(400)

    return 'OK'

@handler.add(MessageEvent, message=TextMessage)
def handle_text_message(event):
    text = event.message.text

    if text == 'こんにちは':
        profile = line_bot_api.get_profile(event.source.user_id)
        line_bot_api.reply_message(
            event.reply_token, [
                TextSendMessage(text='こんにちは' + profile.display_name + 'さん')
            ]
        )

if __name__ == "__main__":
    app.run(port=8000)
```

### ボットの作成

:::message
ここでは詳しく説明はしません。詳しくは[こちら](https://developers.line.biz/ja/docs/messaging-api/getting-started/)をご覧ください。
:::

1. LINE Developersコンソールにログインする
@[card](https://developers.line.biz/console/)

2. 新規プロバイダーを作成する
ホーム画面の［新規プロバイダー作成］をクリックします。

3. チャネルを作成する
作成したプロバイダーページで、［チャネル設定］タブの［Messaging API］をクリックします。

4. チャネルアクセストークンを発行する

### Replitにボットの情報を登録する

作成したreplを開き、右下の`Commands`をタップ  
そしたら`Secret`を開き  
![](https://storage.googleapis.com/zenn-user-upload/7776be36b170-20221008.png)
このようにする  
:::message alert
中身は作成したトークンに入れ替えてください。
:::

### 実行する

`code`に戻り、右下の緑の▶を押すとパッケージのインストールが始まり完了しだい実行されます。(かなり時間がかかる)  
実行されるとサイトが開きます。そのサイトのURLをコピーしておいてください。

### Webhookの設定

作成したチャンネルの`Messaging API設定`>`Webhook設定`>`Webhook URL`にコピーしたURLを貼り付けて最後に`callback`を追加  
例: `https://line-bot.〇〇.repl.co/callback`になってるはず  
そしたら下の`Webhookの利用`をオンにします  

### 友だち追加をして送ってみる

[LINE Official Account Manager](https://manager.line.biz/)に移動し  
@[card](https://manager.line.biz/)
ボットを選択して`友だちを増やす`>`友だち追加ガイド`を開き`URLを作成`で友だち追加をするURLを確認できます。

## 終わりに

初めて記事を書いてみました。説明などが不足しているかもしれませんが何かあったら気軽にコメントください。
