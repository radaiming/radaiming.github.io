---
layout: post
title: "TCP over HTTP"
categories: python
---

现在手上主要的梯子是 http 代理，速度也还行，唯一的问题是 Android 上用起来不方便。ProxyDroid 多年不更新，还要 root，Drony 虽然能工作，但毕竟不是为翻墙设计的，想切换代理都不方便。所以在有人做出来之前只能自己先试试了。对 Android 毕竟不熟，就先在 Linux 上用 Python 先来一遍，先创建 tun 设备，然后路由表把流量指向它，脚本从 tun 设备读，转给 http 代理。嗯说起来比较简单，但踩了很多坑，记几个大的吧。

- 先要避免实现一个 TCP/IP 栈。从 tun 设备读出来的是 raw IP 帧，回写也得是 raw IP 帧，如果想直接处理 TCP 流量，那 TCP 握手、释放、异常处理、连接状态维护什么的都要自己来做，简直吓人。而 [fqrouter](http://fqrouter.tumblr.com/post/51474945203/socks%E4%BB%A3%E7%90%86%E8%BD%ACvpn#_=_) 作者非常聪明地把读到的 IP 帧给 NAT 了一下，使其指向本地的 server，这样就避免了实现一个 TCP/IP 栈的工作。代码就是照着下面的流程图写的，不然自己早混乱了。

<pre>
                                             client request
(2) s: 10.45.39.1:33333                (1) s: 10.45.39.1:33333
    d: 199.59.150.44:443                   d: 199.59.150.44:443
   modify ↓             ↖  ----------  ↙
(3) s: 10.45.39.3:33333  → |   tun   |  ⇆ (4) server listen on 
    d: 10.45.39.1:12345     ----------        10.45.39.1:12345
                         ↙                           ⇅
(5) s: 10.45.39.1:12345                         HTTP CONNECT
    d: 10.45.39.3:33333     ↑                 199.59.150.44:443
    modify ↓               ↗                         ⇅
(6) s: 199.59.150.44:443                         HTTP Proxy
    d: 10.45.39.1:33333
</pre>


- checksum 的重新计算。因为改了 IP 和 port，IP/TCP 的 checksum 都要重新计算。一开始照着维基和别人的实现来做，每次都把整个包给重新加起来，后来发现整个脚本有严重的性能问题，才跑到 500 KB/s 的速度就吃掉我半个多核的 CPU 了，要是放到手机上肯定不行。从 cProfile 结果看 checksum 计算吃了太多 CPU。后来[发现](http://locklessinc.com/articles/tcp_checksum/)其实只需要重新计算改动的那小部分（IP/port）就够了，最后 checksum 计算吞吐量提高了几十倍。

- coroutine or threading? 自从跌跌撞撞学会用 asyncio 开始就基本不考虑用 threading 了，不过在优化完 checksum 计算后感觉 CPU 占用还是太高，至少是[ C 版本](https://github.com/shouya/ip-over-http)的几十倍，就怀疑是不是 asyncio 的问题，于是马上用 threading 改写一遍，CPU 占用又下降了一半多。第二天爬起来又看了眼自己 asyncio 实现，一个 coroutine 里面一个循环完成了上面(4)中和 client/proxy 的交互，随手[改了下](https://github.com/radaiming/tcp-over-http/commit/026efcf77cd80622d340e3a083cf84d56e16c5a7)，多加了一个 coroutine 拆分出另一个读写循环，CPU 占用马上降到和 threading 版本差不多了。也许是之前循环里面不停地 asyncio.wait 吧，而且因为一个循环处理两对读写操作，循环频率并不低。


现在用这脚本做出了翻墙路由，跑下来还算好用，只是性能上还是比 C 版本明显差，可惜并不会写 C。实在不行用 go 写，还能直接编译成 arm binary，方便日后往 Android 上靠。

Update: [go 版本](https://github.com/radaiming/tcp-over-http/blob/master/tcp_over_http.go)已经完成，每秒处理的数据吞吐量比 C 版本高约 3 倍。