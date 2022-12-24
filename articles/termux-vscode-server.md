---
title: "スマホで「VS Code Server」を建てる方法"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", termux]
published: true
---

# 今回使用するもの

- VS Code Server
- Android
- Termux

## 「VS Code Server」とは

@[card](https://code.visualstudio.com/blogs/2022/07/07/vscode-server)

## Termuxとは

AndroidでLinuxを動かすためのアプリです。

@[card](https://github.com/termux/termux-app)

# 「VS Code Server」を起動するまで

## 1.Termuxのインストール

### Githubからの場合

@[card](https://github.com/termux/termux-app/releases)

![](<https://storage.googleapis.com/zenn-user-upload/88a4d9a450d1-20221003.png> =250x)
**termux-app_v0.118.0+github-debug_universal.apk**をダウンロードし、インストール

:::details その他の方法

### F-droidからの場合

@[card](https://f-droid.org/en/packages/com.termux/)
F-droidをインストールしたあとにTermuxをインストール
:::
---

インストールが完了したら起動します。
![](<https://storage.googleapis.com/zenn-user-upload/9dbdf59b3166-20221003.png> =250x)

## 2.TermuxにUbuntuを入れる

Ubuntuをインストールするために必要なパッケージのインストール

```sh
pkg install wget openssl-tool proot
```

次に、Ubuntuをインストールし、自動で起動するようにします

```sh
wget https://raw.githubusercontent.com/EXALAB/AnLinux-Resources/master/Scripts/Installer/Ubuntu/ubuntu.sh && bash ubuntu.sh && echo "./start-ubuntu.sh" >> ~/.bashrc && ./start-ubuntu.sh
```

:::details Ubuntuを日本語化する

Ubuntuを起動したあとに以下のコマンドを実行

```sh
apt install language-pack-ja -y && echo 'export LANG=ja_JP.UTF-8' >> ~/.bashrc && echo 'export LANGUAGE="ja_JP:ja"' >> ~/.bashrc
```

:::

## 3.「VS Code Server」のインストール

```sh
wget -O- https://aka.ms/install-vscode-server/setup.sh | sh
```

インストールが正常にできると

```sh
code-server -h
```

使い方を確認できます。

## 4.起動の仕方

```sh
code-server serve-local
```

を実行すると起動でき、
![](<https://storage.googleapis.com/zenn-user-upload/a411109a708d-20221003.png> =250x)
ライセンスに同意すると
`Web UI available at http://localhost:8000/?tkn=***************`と表示されます。
このトークンを含めたURLにアクセスするとVS Codeが起動します。
