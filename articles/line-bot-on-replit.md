---
title: "LineBotをスマホとReplitで作ってみる"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linebot", "python", "replit"]
published: false
---

## 今回使用するもの

- Python
- [line-bot-sdk-python](https://github.com/line/line-bot-sdk-python)
- [Flask](https://github.com/pallets/flask)
- Replit(Pythonを動かすために)

## 準備

1. まずはReplitのアカウントを作成

@[card](https://replit.com/)

2. Pythonのreplを作成する

![](/images/line-bot-on-replit/replit-create-python.png)

3. main.pyにコードを貼り付ける

```py
import os
import sys
from argparse import ArgumentParser

from flask import Flask, request, abort
from linebot import (
    LineBotApi, WebhookParser
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
    except LineBotApiError as e:
        print("Got exception from LINE Messaging API: %s\n" % e.message)
        for m in e.error.details:
            print("  %s: %s" % (m.property, m.message))
        print("\n")
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
