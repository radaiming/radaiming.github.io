---
layout: post
title: "用 Telegram 做 push 通知"
categories: misc
---

Telegram 实在是太好用了，拿来做 push 通知自然也是坠吼滴，相比之下 Pushbullet 的移动 app 实在太重，又没有原生客户端。

在 Telegram 里直接和 [@BotFather](tg://resolve?domain=BotFather) 对话建立一个新的 bot，它会给出一个 token。要让 bot 给自己发消息的话还得有会话 ID，自己给 bot 发一条消息，再用 API 替 bot 收这条消息的 json 就可以看到。要让 bot 给自己发消息，只需要一个简单的 GET 里带上 token 和会话 ID 就够了。

不过这个 bot 开了这么久，发现自己并不需要什么通知，毕竟不是运维。
