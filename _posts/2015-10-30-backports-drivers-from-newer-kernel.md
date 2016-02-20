---
layout: post
title: "用 Backports 为旧 kernel 引入新驱动"
categories: linux
---

这篇拖了好几个月，主要是碰到的问题折腾了好久，最后离职前也并没很好地解决，于是这篇也是写得虎头蛇尾。

当时的情景是，我们产品需要停留在 kernel 3.8.0，并且要支持很多种 wifi 卡，所以一直依赖于 Backports。后来有张新的 wifi 卡用的是不在 kernel tree 里的 [8812au.ko](https://github.com/abperiasamy/rtl8812AU_8821AU_linux)，这本身没什么问题，编好后跑得也比较稳定，但有个问题，如果我们用了 Backports，由于它替换掉了 mac80211.ko 和 nl80211.ko，而 8812au 又是基于 nl80211 的，所以就加载不上了。我们这并没有能做 kernel 编程的，所以把 8812au 给 merge 到 kernel 或者 Backports 都没法做。Google 了好久，看到篇挂载 kernel.org 下但是写得很烂很不好懂的[文档](https://backports.wiki.kernel.org/index.php/Documentation/integration)提到可以把 Backports 的变动 apply 到旧版本的 kernel 源码上，得到一份带着新驱动的源码。按理来说，这么一来既可以通过这新的源码来使用新的驱动，又可以针对其编译 8812au。Backports 仓库里面有个 [gentree.py](https://git.kernel.org/cgit/linux/kernel/git/backports/backports.git/tree/gentree.py)，文档说是这么用的：

~~~~~~~~
 $./gentree.py --integrate --clean --gitdebug --git-revision next-20141114 ~/linux-next/ ~/linux/
~~~~~~~~

其中 linux-next 得是个 git repo，next-20141114 是 linux-next 里的一个 tag，最后的 linux 目录里是 port 的目标代码目录（我这里就是旧的 kernel 3.8.0）。这脚本很容易报错，又没什么文档，不得不经常自己跑 pdb 来看原因。成功的输出大概是这样的：

~~~~~~
$ ./gentree.py --clean --integrate --gitdebug --git-revision v4.1 ../linux-stable-v4.1/ ~/tmp/linux-source-3.8.0/
Get original source files from git ...
Applying patches from patches to /home/aarondai/tmp/linux-source-3.8.0/backports/ ...
Modify Kconfig tree ...
Rewrite Makefiles and Kconfig files ...
Applying patches from integration-patches/ to /home/aarondai/tmp/linux-source-3.8.0/ ...
Done!
~~~~~~

接下来 make menuconfig 的时候就会看到多了个选项：

~~~~~~
[ ] Backport Linux v4.1-0-gb953c0d (backports v4.1.1-1-0-g8286954)  --->
~~~~~~

最后编译的时候还碰到缺头文件的报错，从 Backports 里面 copy 出来后编过。别的编译错误也碰到过，后来通过调整 kernel 配置解决的。由于只需要模块，于是：

~~~~~~~~
make -j 10 SUBDIRS=backports modules
~~~~~~~~

最后编 8812au。嗯，都编出来了，但是加载的时候报错，当时具体什么报错，是 8812au 还是 Backports 驱动报错，都不记得了，后来都没（时间）搞定这个问题。
