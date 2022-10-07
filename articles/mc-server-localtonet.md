---
title: "localtonetを使ってみた"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["minecraft", "minecraftbedrock", "localtonet"]
published: false
---

今回は、localtonetを使いをMincraft統合版のサーバーをローカルからインターネットに公開する方法を紹介します。

## localtonetとは

https://localtonet.com/
簡単にいうと、ngrokのようなローカルからインターネットに公開するツールです。
`HTTP`, `HTTPS`, `TCP`, `UDP`(←ここが**重要**)の4つのプロトコルに対応しています。
さらに、localtonetは、様々なOSに対応しています。(Windows, macOS, Linux, Android, Docker)
しかし、ngrokは無料で使えるのに対し、localtonetは**一部有料**です。
Minecrat統合版のサーバーを公開するには、`TCP`と`UDP`の2つのプロトコルが必要です。

:::message
プランについては、[こちら](https://localtonet.com/#pricing)を参照してください。
:::

## localtonetのインストール

localtonetは、様々なOSに対応しています。(Windows, macOS, Linux, Android, Docker)
今回は、Linuxにインストールしてみます。

```bash
wget https://localtonet.com/download/localtonet-linux-x64.zip
unzip localtonet-linux-x64.zip
```

インストールが完了したら、`localtonet`に権限を付与します。

```bash
chmod 777 localtonet
```

これで、localtonetのインストールは完了です。

### アカウントの作成

localtonetを使うには、アカウントが必要です。
https://localtonet.com/ にアクセスして、アカウントを作成します。
DashBoardにアクセスすると、`My Default AuthToken`が表示されます。
これをコピーしておきます。

## tunnelの作成

localtonetを使うには、tunnelを作成する必要があります。
tunnelは、`TCP`と`UDP`の2つのプロトコルが必要です。
https://localtonet.com/tunnel/tcpudp にアクセスして、tunnelを作成します。
`Protocol Type :`には、`UDP - TCP`を選択し、`Port :`には、`19132`を入力します。
`Server:`は何でも大丈夫です。
![](https://storage.googleapis.com/zenn-user-upload/dab1be2f2415-20221007.png)
`create`をクリックすると、トンネルが作成され、下に作成したトンネルの情報が表示されます。

## localtonetの起動

localtonetを起動するには、`localtonet`コマンドを実行します。

```bash
./localtonet
```

起動すると、`Auth Token`を聞かれるので、先ほどコピーした`My Default AuthToken`を入力します。
これで、localtonetの起動は完了です。

## Minecraft統合版のサーバーのインストール＆起動

Minecraft統合版のサーバーをインストールします。

```bash
wget https://minecraft.azureedge.net/bin-linux/bedrock-server-1.19.31.01.zip
unzip bedrock-server-1.19.31.01.zip
```

これで、Minecraft統合版のサーバーのインストールは完了です。
次に、Minecraft統合版のサーバーを起動します。

```bash
LD_LIBRARY_PATH=. ./bedrock_server
```

これで、Minecraft統合版のサーバーの起動は完了です。

## Minecraft統合版のサーバーの公開

先ほどの作成したtunnelの`more`をクリックすると、`Start`ボタンが表示されます。
![](https://storage.googleapis.com/zenn-user-upload/84a9639d5d1b-20221007.png)
これをクリックすると、tunnelが開始され、`Stop`ボタンが表示されます。
これで、Minecraft統合版のサーバーの公開は完了です。

:::message
`Please Start or Update localtonet app!`と表示された場合は、localtonetを起動した状態で、開始してください。
:::

## Minecraft統合版のサーバーへの接続

![](https://storage.googleapis.com/zenn-user-upload/d9923553826e-20221007.png)
`サーバー アドレス`に`Server Domain`を入力します。
`ポート`には、`Server Port`を入力します。
名前は何でも大丈夫です。
これで、Minecraft統合版のサーバーへの接続は完了です。
:::details ちなみに
codespacesで立てることもできました。
![](https://storage.googleapis.com/zenn-user-upload/3082cde7805e-20221007.png)
:::

![](https://storage.googleapis.com/zenn-user-upload/f3cc10c2259a-20221007.png)
じっさいに、Minecraft統合版のサーバーへ接続してみました。

## まとめ

今回は、localtonetを使って、Minecraft統合版のサーバーを公開する方法を紹介しました。
localtonetは、一部有料ですがlocaltonetはUDPをサポートしているので、ちょっとしたサーバーを公開するには、localtonetを使うのが一番簡単です。
