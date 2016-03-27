---
layout: post
title: "彻底解决 DNS 污染问题"
categories: python
---

这是篇流水帐。

因为一般都在用 https 代理做梯子，所以大部分时候是不介意 DNS 污染的，但也不能放任不管，指不定哪天就被坑了。最早用的是 goagent 里面带的 DNS 缓存脚本，后来用过 ChinaDNS，弃用的原因也不记得了。至于 dnscrypt_proxy，因为出国 UDP 包经常被丢的缘故，相当不稳定。然后试过让 Pcap_DNSProxy 走 HTTP CONNECT 代理，虽然能工作，但这延时实在太大了，而且 Pcap_DNSProxy 文档实在太少。

在找别的能走代理的替代的时候看到了 [Tcp-DNS-proxy](https://github.com/henices/Tcp-DNS-proxy)，看了眼源码，原来 DNS 代理只需要把收到的 UDP 包内容直接加个 TCP 头就可以发出去了，实在简单，于是就开始自己写一个类似 DNS relay 的东西，client 端收到请求后直接把 UDP 包加密然后发给 server，server 解密后把 UDP 包发给 8.8.8.8，再把收到的 UDP 回应加密发回给 client，client 解密然后发回给查询者，就这么简单。Tcp-DNS-proxy 里面用的是 ThreadingMixIn，而我老是心理上不爽线程的开销（虽然自己的机器每秒查询量撑死不过 10 个），自然选择了 asyncio。话说 asyncio 上手比 gevent 难不少，几个月前第一次看 文档/sample/PEP 的时候看得云里雾里，使用的过程简直就是一个试错的过程，跌跌撞撞给写出来了 [dns_tunnel.py](https://github.com/radaiming/DNS_Tunnel/blob/e1bde904dc9daa63b05bf8ed67f710f6e9553930/dns_tunnel.py)，欣慰的是能 work，也大概会用 asyncio 了。但到了晚上就发现该死的电信开始狂丢 UDP 包了，这脚本完全没法用。

在写 UDP 版本的时候就想起 Heroku 貌似支持 raw socket 什么的，搜了下看到的主要是说支持 WebSocket。在发现 UDP 包被狂丢后目光自然转向了 WebSocket。试了好几个库，好些是为 web 服务做了些封装的，而我只想建立一个 WebSocket 连接，然后发包收包而已。最后用了 [WebSockets](https://github.com/aaugustin/websockets) 这个略底层的库，简直赞。因为自己服务器和 Heroku 有 https 加持，所以我加密都不用做了，连接直接走 wss 协议。最后试下来效果相当不错，查询延时和 ping 服务器的延时基本一致，wss 连接保持个几十小时完全没问题，excited！

多扯两句 Heroku 和 OpenShift 的对比，我这脚本没 wsgi，直接起起来就开始监听端口，就我简单的使用体验来说，Heroku 基本完胜，支持的 Python 版本丰富，没有 wsgi 的可以 Procfile 文件直接指定启动命令，随手 Google 一会就跑起来了，只是免费版每天有运行时间限制，以及估计是网络质量问题，wss 连接没有到我 vps 那么稳定；而 OpenShift 除了提供 shell 外别的都比较蛋疼，Python 最高只有 3.3，没有 wsgi 还只能选 diy 类型的服务，自己写 hook 脚本去 build 好环境（主要是 Python 3），折腾半天就算了，最后还跑不起来，因为 OpenShift 只允许绑定特定端口，这样的话我没法用 socket 往外发 UDP 包了，一 bind 就报 permission denied。

墙越来越高，手上的梯子慢慢不那么好用了，既然贫贱不能移那就自己动手吧。
