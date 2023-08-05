---
title: "ConoHaをDiscordから起動したい"
emoji: "🎐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["conoha", "discord", "arksurvivalevolved"]
published: true
---

# はじめに

最近、友達のすすめで「ARK：Survival Evolved」というゲームを始めました。
その際に、ConoHa の VPS を使ってマルチプレイサーバーを立てたのですが、起動する際に毎回 ConoHa のダッシュボードにログインして起動しないといけなかったので、Discord からできるようにしました。

# 開発環境

- TypeScript 5.1.6
- discord.js 14.12.1

```bash
pnpm add -D typescript @types/node
pnpm add discord.js dotenv zod
```

:::message
この記事ではサーバーは立ててあることを前提として話を進めていきます。
ConoHa だとすでに ARK Server のイメージが用意されているので簡単に立てることができます。
:::

# 環境変数の設定

zod を使用して環境変数を設定します。

```ts:lib/env.ts
import 'dotenv/config'
import { z } from 'zod'

export const envSchema = z.object({
  DISCORD_TOKEN: z.string(),
  DISCORD_CLIENT_ID: z.string(),
  CONOHA_USERNAME: z.string(),
  CONOHA_PASSWORD: z.string(),
  TENANT_ID: z.string(),
  SERVER_ID: z.string(),
})

export const env = envSchema.parse(process.env)
```

# ConoHa API

ConoHa の API を使ってサーバーを起動していくのですが、調べてもあまり情報がなかったので、公式のドキュメントを見ながらやってきます。

https://www.conoha.jp/docs/

## APIを使うための準備

まず、API を使うために ConoHa のダッシュボードにアクセスし、APIユーザーを作成する必要があります。
APIユーザーを作成したら、`.env`に書き込んでいきます。
また、テナントIDが必要になるのでAPIユーザーの上にあるテナント情報から確認してください。

```env
CONOHA_USERNAME=APIユーザーのID
CONOHA_PASSWORD=APIユーザーのパスワード
TENANT_ID=テナントID
```

次にサーバーIDを **サーバー>VPS設定>UUID** からコピーしてください。

```env
...
SERVER_ID=サーバーのUUID
```

### 今回使用するAPI

今回は、サーバーの起動と停止をするため、以下の API を使用します。

- **Identity Service**
  - トークンの発行

https://www.conoha.jp/docs/identity-post_tokens.php

サーバーを起動したり停止するためには、ここで発行するトークンが必要になります。

- **Compute Service**
  - サーバーの起動

    @[card](https://www.conoha.jp/docs/compute-power_on_vm.php)

  - サーバーの停止

    @[card](https://www.conoha.jp/docs/compute-stop_cleanly_vm.php)

  - サーバーの再起動

    @[card](https://www.conoha.jp/docs/compute-reboot_vm.php)

  - サーバーの状態を取得

    @[card](https://www.conoha.jp/docs/compute-get_vms_detail_specified.php)

## APIの実装

### エンドポイントの設定

各APIのエンドポイントはサーバーによって異なるのではじめの方に設定しておきます。
**API>API情報>エンドポイント** から確認できます。

```ts:lib/conoha.ts
import { env } from './env'

const URL_END_POINT_IDENTITY = 'https://identity.tyo2.conoha.io/v2.0/'
const URL_END_POINT_COMPUTE = 'https://compute.tyo2.conoha.io/v2/' + env.TENANT_ID
```

### トークンの発行

```ts:lib/conoha.ts
export async function getToken() {
  const res = await fetch(URL_END_POINT_IDENTITY + 'tokens', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      auth: {
        passwordCredentials: {
          username: env.CONOHA_USERNAME,
          password: env.CONOHA_PASSWORD,
        },
        tenantId: env.TENANT_ID,
      },
    }),
  })
  const json = await res.json()
  return json.access.token.id
}
```

トークンを発行するにはAPIユーザーのIDとパスワードが必要になります。

### サーバーの起動と停止、再起動

```ts:lib/conoha.ts
export const doAction = async (action: 'start' | 'stop' | 'reboot') => {
  const token = await getToken()
  if (!token) throw new Error('Token not found')
  const status = await getStatus()
  if (action == 'start' && status == 'ACTIVE') throw new Error('すでに起動しています')
  if (action == 'stop' && status == 'SHUTOFF') throw new Error('すでに停止しています')
  if (action == 'reboot' && status == 'SHUTOFF') throw new Error('停止しているため再起動できません')
  let body = {}
  if (action == 'start') {
    body = {
      'os-start': null,
    }
  } else if (action == 'stop') {
    body = {
      'os-stop': null,
    }
  } else if (action == 'reboot') {
    body = {
      reboot: {
        type: 'SOFT',
      },
    }
  }
  const res = await fetch(URL_END_POINT_COMPUTE + '/servers/' + env.SERVER_ID + '/action', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Auth-Token': token,
    },
    body: JSON.stringify(body),
  })
  if (!res.ok) {
    const json = await res.text()
    throw new Error(json)
  }
}
```

起動するときと停止するときのリクエスト先は同じで、ボディの中身でアクションが変わります。
すでに起動している状態で起動するとエラーが返ってくるので防ぐためにサーバーの状態を取得しています。

### サーバーの状態を取得

```ts:lib/conoha.ts
export const getStatus = async () => {
  const token = await getToken()
  if (!token) throw new Error('Token not found')
  const res = await fetch(URL_END_POINT_COMPUTE + '/servers/' + env.SERVER_ID, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'X-Auth-Token': token,
    },
  })
  if (!res.ok) {
    const json = await res.text()
    throw new Error(json)
  }
  const json = await res.json()
  return json.server.status
}
```

# Discord Bot

:::message
ボットのトークンとクライアントIDは取得している前提で話を進めていきます。
:::

## ボットの作成

今回は、discord.js を使用してスラッシュコマンドで操作できるボットを作成します。

### 環境変数の設定

```env
...
DISCORD_TOKEN=ボットのトークン
DISCORD_CLIENT_ID=ボットのクライアントID
```


### スラッシュコマンドを登録

登録するコマンド

- **start** : サーバーを起動
- **stop** : サーバーを停止
- **reboot** : サーバーを再起動
- **status** : サーバーの状態を取得

```ts:registerCommand.ts
import { REST, Routes } from 'discord.js'
import { env } from './lib/env'

const commands = [
  {
    name: 'start',
    description: 'サーバーを起動する',
  },
  {
    name: 'stop',
    description: 'サーバーを停止する',
  },
  {
    name: 'reboot',
    description: 'サーバーを再起動する',
  },
  {
    name: 'status',
    description: 'サーバーの状態を確認する',
  },
]

const rest = new REST({ version: '10' }).setToken(env.DISCORD_TOKEN)

async function refreshCommands() {
  try {
    console.log('Started refreshing application (/) commands.')

    await rest.put(Routes.applicationCommands(env.DISCORD_CLIENT_ID), { body: commands })

    console.log('Successfully reloaded application (/) commands.')
  } catch (error) {
    console.error(error)
  }
}

refreshCommands()
```

### ボットの実装

```ts:bot.ts
import { ActivityType, Client, GatewayIntentBits } from 'discord.js'
import { doAction, getStatus } from './lib/conoha'
import { env } from './lib/env'

const client = new Client({ intents: [GatewayIntentBits.Guilds] })

client.on('ready', () => {
  console.log(`Logged in as ${client.user!.tag}!`)
  setInterval(function () {
    getStatus().then((status) => {
      if (status == 'ACTIVE') {
        client.user!.setStatus('online')
        client.user!.setActivity(`サーバーは起動中`, { type: ActivityType.Playing })
      }
      if (status == 'SHUTOFF') {
        client.user!.setStatus('idle')
        client.user!.setActivity(`サーバーは停止中`, { type: ActivityType.Playing })
      }
    })
  }, 5000)
})

client.on('interactionCreate', async (interaction) => {
  if (!interaction.isChatInputCommand()) return

  if (interaction.commandName === 'start') {
    const message = await interaction.reply('サーバーを起動しています...')
    try {
      await doAction('start')
    } catch (e: any) {
      message.edit(`❌サーバーの起動に失敗しました\n${e.message}`)
      return
    }
    message.edit('✅サーバーを起動しました\n参加できるようになるまで数分かかる場合があります')
  } else if (interaction.commandName === 'stop') {
    const message = await interaction.reply('サーバーを停止しています...')
    try {
      await doAction('stop')
    } catch (e: any) {
      message.edit(`❌サーバーの停止に失敗しました\n${e.message}`)
      return
    }
    message.edit('✅サーバーを停止しました')
  } else if (interaction.commandName === 'reboot') {
    const message = await interaction.reply('サーバーを再起動しています...')
    try {
      await doAction('reboot')
    } catch (e: any) {
      message.edit(`❌サーバーの再起動に失敗しました\n${e.message}`)
      return
    }
    message.edit('✅サーバーを再起動しました')
  } else if (interaction.commandName === 'status') {
    const status = await getStatus()
    let statusText = ''
    if (status == 'ACTIVE') statusText = '起動中🟢'
    else if (status == 'SHUTOFF') statusText = '停止中🔴'
    await interaction.reply('サーバーの状態は' + statusText + 'です')
  }
})

client.login(env.DISCORD_TOKEN)
```

`ready`のイベントで5秒ごとにサーバーの状態を取得して、ボットのステータスを変更しています。
`interactionCreate`のイベントでスラッシュコマンドの処理をしています。

# 実行

```bash
pnpm tsc **/*.ts --outDir dist # TypeScriptをコンパイル
node dist/registerCommand.js # スラッシュコマンドの登録
node dist/bot.js # ボットの起動
```

# おわりに

Discord からサーバーを起動できるようになったので、自分で起動する必要がなくなったし、使う時だけ起動できるのでコストも抑えられるようになりました。
今回は紹介していませんが、Steam の API を使ってサーバーのオンラインプレイヤー数も表示できました。

https://github.com/HRTK92/ConoHa-Bot
