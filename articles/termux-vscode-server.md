---
title: "スマホで「VS Code Server」を建てる方法"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", termux]
published: true
---

# 今回使用するもの

- VS Code Server
- Termux

## 「VS Code Server」とは

@[card](https://code.visualstudio.com/blogs/2022/07/07/vscode-server)

## Termuxとは

@[card](https://github.com/termux/termux-app)

# 「VS Code Server」を起動するまで

## 1.Termuxのインストール

### Githubからの場合

@[card](https://github.com/termux/termux-app/releases/tag/v0.118.0)
ここから最新のTermuxをAPKファイルをダウンロードし、インストール

### F-droidからの場合

@[card](https://f-droid.org/en/packages/com.termux/)
F-droidをインストールしたあとにTermuxをインストール

## 2.TermuxにUbuntuを入れる

必要なパッケージのインストール
```sh
pkg install wget openssl-tool proot
```

Ubuntuをインストールし、自動で起動するようにする
```sh
wget https://raw.githubusercontent.com/EXALAB/AnLinux-Resources/master/Scripts/Installer/Ubuntu/ubuntu.sh && bash ubuntu.sh && echo "./start-ubuntu.sh" >> ~/.bashrc && ./start-ubuntu.sh
```

:::details 日本語化する


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
使い方が見れるようになると思います

## 4.起動の仕方

```sh
code-server serve-local
```
ホストやポートを指定しない場合は`localhost:8000`で起動します
