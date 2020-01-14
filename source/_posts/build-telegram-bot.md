---
title: 如何创建一个 Telegram Bot
date: 2019-04-20 10:25:50
categories:
  - Unix
tags:
  - Telegram
---

作为一个 Telegram 的重度用户，早在使用之初我就听说 `Telegram Bot` 的开放性。深入了解以后我使用 Bot 实现了一系列功能，比如通过 Bot 查询公交到站情况、查询历史上的今天、通过 `Github Release API` 获取最新 SS 下载链接以及查询信息生成二维码等。这些功能通过简单命令调用，相当方便。今天就来分享一下如何创建一个 `Telegram Bot`。

<!-- more -->

### 准备工作

- 通过 [Bot Father](https://telegram.me/botfather) 填写详情并获取对应的 Token

- 熟悉官方文档，参见：[Telegram Bot API](https://core.telegram.org/bots/api)

- 加持 HTTPS 的域名以及一台 VPS 用来运行你的实例

### 正式开发

官方提供的文档已经十分详细了，但是很多细节需要二次封装，所以使用了基于 Node.js 的 `node-telegram-bot-api`，以下为了方便，简称 NT。

NT 的使用相当简单，只需要十多行代码就可以实现一个简单的 Bot。这里贴一段官方的代码，其中的注释我已经翻译过来了。

```Javascript
const TelegramBot = require('node-telegram-bot-api')

// 替换 token 的值为你从 @BotFather 那里申请的 token
const token = 'YOUR_TELEGRAM_BOT_TOKEN'

// 由于是在开发阶段，所以使用 'polling'（轮询）模式来获取发给 bot 的信息
const bot = new TelegramBot(token, { polling: true })

// 匹配 '/echo [whatever]'
bot.onText(/\/echo (.+)/, (msg, match) => {
  // 'msg' 是从 Telegram 接手到的信息
  // 'match' 是对传来的消息执行正则后获得的结果

  const chatId = msg.chat.id
  const resp = match[1] // 捕获到的 'whatever'

  // 把匹配到的 'whatever' 发送回对话中
  bot.sendMessage(chatId, resp)
})

// 监听任何种类的信息，如：音频，视频，图片，文本等
bot.on('message', msg => {
  const chatId = msg.chat.id

  // 发送 'Received your message' 到对话中
  bot.sendMessage(chatId, 'Received your message')
})
```

Telegram 的工作方式分为两种，一种是不停的轮询 Telegram 服务器来获取发送给你 Bot 的消息，即 polling。另一种则是设置 WebHook ，这样当 Telegram 服务器收到变动则会将消息内容及相关信息通过 WebHook 交给你的服务端来处理，但是这种方式需要你的 VPS 配置 HTTPS 才能使用（安全考量）。

关于 HTTPS 的认证，可以看看 [acme.sh](https://github.com/Neilpang/acme.sh)，它通过实现 acme 协议，可以从 letsencrypt 生成免费的证书，并且提供定时续签等功能，十分的方便。

当你完成 HTTPS 认证后，可以通过 `process.env.NODE_ENV` 来实现开发和线上环境的区分，如果你还不了解 `process.env`，请看[这里](https://nodejs.org/api/process.html#process_process_env)。

```Javascript
if (process.env.NODE_ENV === 'development') {
  // 开发环境使用轮询模式
  const bot = new TelegramBot(TOKEN, { polling: true })
} else if (process.env.NODE_ENV === 'production') {
  // 线上环境设置 `WebHook`
  const bot = new TelegramBot(TOKEN)
  bot.setWebHook(VPSURL)

  // 将接收到的数据转成 JSON
  const app = express()
  app.use(bodyParser.json())

  // 配置路由接受来自 Telegram 的消息
  app.post(`/bot`, (req, res) => {
    bot.processUpdate(req.body)
    res.sendStatus(200)
  })

  // 启动 Express 服务器
  app.listen(9999, () => {
    console.log('Express server is listening on 9999')
  })
}
```

当完成所有步骤后，你就可以将其部署在你的 VPS 之上了，这里推荐使用 PM2 来实现部署，不仅支持负载均衡，还能实现热重启。这里贴一段 PM2 配置文件，使用 `pm2 start app.json` 就可以顺畅体验了。

```Javascript
{
  "apps": [
    {
      "name": "TelegramBot",
      "script": "./server/index.js",
      "env": {
        "NODE_ENV": "production"
      }
    }
  ]
}
```
