---
layout: post
title: "把 μTorrent 关 docker 里"
categories: linux
---

一般来说可以用 transmission/deluge 下载 BT，可是对我来说有个问题就是，我选择不要下载的文件它也会给创建出来。transmission 的[理由](https://trac.transmissionbt.com/ticket/514)是 BT 里面一个分片可能是跨文件的，所以要下载其中一个文件的话，另一个文件也得给我创建出来。然后终于有人提到 μTorrent 并不创建文件，而是把分片里跨文件的部分都保存到另一个单独的文件里。接着[八年](https://trac.transmissionbt.com/ticket/532)过去了，这功能都没有合到主干。

但是 μTorrent 的话，毕竟是闭源的，被害妄想一发作就想把它 jail 起来，虽说 docker 安全性貌似也不好。

把 μTorrent 下载下来后解压好，Dockerfile:

~~~~~~~~
FROM debian:7
RUN apt-get update
RUN apt-get install -y apt-utils libssl1.0.0 locales
RUN useradd -m -s /bin/bash ut
RUN echo 'en_US.UTF-8 UTF-8' > /etc/locale.gen
RUN locale-gen
ENV LC_ALL=en_US.UTF-8
USER ut
EXPOSE 9099
VOLUME ["/data"]
CMD ["/data/.ut/utorrent-server-alpha-v3_3/utserver", "-configfile", "/data/.ut/utorrent-server-alpha-v3_3/utserver.conf", "-logfile", "/data/.ut/utorrent-server-alpha-v3_3/utserver.log", "-settingspath", "/data/.ut/utorrent-server-alpha-v3_3/"]
~~~~~~~~

如果没有 UTF-8 的 locale 印象中对中文文件名处理有问题。其实后来曾经尝试过 Alpine Linux，毕竟小巧下载快，但完全跑不起来。

当然我喜欢用 systemd 来起 utorrent.service:

~~~~~~~~
[Unit]
Description=utorrent
After=docker.service

[Service]
ExecStartPre=/bin/sh -c "/usr/bin/docker rm ut || :"
ExecStart=/usr/bin/docker run --rm --name ut --cpuset-cpus=0 -p 127.0.0.1:9099:9099 -v /media/bt/:/data/ radaiming:ut-debian
Restart=always
RestartSec=10
ExecStop=/usr/bin/docker stop -t 20 ut
TimeoutStopSec=25

[Install]
WantedBy=multi-user.target
~~~~~~~~

印象中 BT 客户端在退出的时候需要通知 tracker/peer 自己下线，经常要花费不少时间，所以给 docker stop 加了 20 秒的超时。μTorrent 有 bug，有时会莫名吃满 CPU，所以当时限制只允许用一个核避免影响太大。9099 是 web 控制界面的端口。就这么跑了至少半年多，感觉并没有什么问题。
