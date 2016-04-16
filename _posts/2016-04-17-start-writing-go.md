---
layout: post
title: "开始写 Go"
categories: go
---

好几年前就尝试过 Go，但一直以来 Python 对我而言够用也足够好用，所以就算 Go 有些不错的特性也被我无视了，直到最近碰到的几个问题，才让我打起了 Go 的主意：

- 性能问题，Python 这边我优化了挺久，[性能都比 C 差太多](https://radaiming.github.io/python/2016/04/14/tcp-over-http.html)，而我又很不想写 C；

- 跟编译出来的二进制相比，Python 解释器启动太过缓慢，这是人可察觉到的慢，有时候实在是难以忍受；

- Go 写的代码默认是静态编译的，能方便地为 Android 编译，而 Python 那边，呃我还没怎么去研究 [python-for-android](https://python-for-android.readthedocs.org)，感觉看起来比较麻烦。

走马观花似地过了一遍 Go by Example 后就[开始写](https://github.com/radaiming/misc/blob/master/go/bt_search/btsow.go)了，至少目前写起来比写 C 快且舒服多了。