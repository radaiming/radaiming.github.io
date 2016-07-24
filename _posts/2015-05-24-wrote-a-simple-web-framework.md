---
layout: post
title: "写了个简单的 web 框架"
categories: go
---

其实这是给我的面试题，刚接到的时候很懵，以前只简单地用过 Flask，然而要自己实现一个一下子不知道怎么下手了。想了挺久想明白了大概应该怎么做：建立一个全局 map 用来存放用户自定义的 function，map 是两级的，分别用 URL 和 method 做 key，然后提供一个 AddRoute function 来让用户往指定的 URL 和 method 上添加 function；另一边用 net 库监听本地 TCP 端口，收到请求后解析收到的数据，然后根据请求 URL 和 method 喂给用户传进来的对应的 function，拿到 function 返回值后组装好 TCP 数据写回去就行。

在尝试开始做之后发现 Go 的 http 库已经封装好了很多东西，完全不用自己解析 TCP 数据，晚上完成了[第一个 commit](https://github.com/radaiming/dumpling/commit/c7226c5c78b5a4821ec6767d57da76a6aa463927)，现在看来非常简单，但其实是所有 commit 里面最费力的之一，跑起来的时候同时有三种感觉：“work 了！”，“还真 work”，“果然 work”。

然而这时候用户添加的 function 只能 return HTTP content，而不能自定义 status code 和 header 什么的，于是对用户定义的 function 的返回值做了[修改](https://github.com/radaiming/dumpling/commit/f1414333126a64b664164dc82089955ad91d9a94)，需要同时返回 status code，header 和 content。

这么做当然 work，但对用户其实很繁琐，也很丑，而且写成这个样子，预感之后添加新功能会比较困难。route 方面篓子建议我直接实现 Get 和 Post 这样的 method，方便自动补全，而我也感觉调 AddRoute 要传三个参数也确实有点啰嗦。第二天翻了翻 [goat](https://github.com/bahlo/goat) 的源码，对 route 做了[改进](https://github.com/radaiming/dumpling/commit/171866a6810894349b3675e443449cc69b4fe580)。虽然不太想，但写得还是跟人家的代码比较像了。晚上的时候又参考别的框架，添加了一个 HTTPContext struct，用于存放当前 HTTP 连接相关的信息，然后暴露 SetStatusCode AddHeader Response 等 method 给用户调用，这时候写出的 [sample](https://github.com/radaiming/dumpling/blob/e6acc48657bf7c4836cdc6ef76d0d5fc6fc7482b/samples/redirect.go) 就比[之前的](https://github.com/radaiming/dumpling/blob/4af4911d2decfab62fde67998cd383b840803be4/samples/redirect.go)好看多了。

同一天还实现了对 middleware 的支持，现在看来简单，其实就是一系列返回值为 http.Handler 类型的函数串起来，然后 http.ListenAndServe 就相当于是在一层层地往里调 ServeHTTP(w ResponseWriter, r *Request) 了。但当时在理解的时候依旧费劲和迷惑，比如 goat 里面把 middlewares [chain 起来](https://github.com/bahlo/goat/blob/4de72452b8cfe7dc1dd9acabf3ca8d4d443dd1ad/middleware.go#L15)的方式，看起来并不能支持 [gorilla 的 middleware](https://godoc.org/github.com/gorilla/handlers)，因为后者很多 middleware 是需要传入多个参数的，这导致我长时间不知道 middleware 到底怎么用的，在想通后决定让用户自己 chain 好了再传进来，也不知道算不算偷懒。而这篇 [Writing HTTP Middleware in Go](https://justinas.org/writing-http-middleware-in-go/) 则大大帮助了我对 middleware 的理解。

以上做完后这个小框架核心的部分基本就 OK 了，剩下的一些 TODO，比如解析 URL 和 POST body，test，doc，都不是太陌生的东西，大概能想到该怎么做，而且 Go 基础库本身就封装好了太多东西，花时间去撸代码就行了，除了在“支持” multipart/form-data 上。

在对 POST 请求做封装的时候了解到，浏览器对 POST 是一定要支持 application/x-www-form-urlencoded 和 multipart/form-data 的，前者好理解和处理，而后者相对复杂一些，一开始的时候选择[暴露](https://github.com/radaiming/dumpling/commit/7c359d2ea529e632e292f73d27094efb541bdc48) MultipartReader 出去，让用户自己调 multipart.Reader 的 NextPart 一个个文件处理。后来（其实就是写到这的时候）还是觉得有点丑，而且发现 NextPart 遍历完所有 part 后不能从头再遍历一次，意味着用户上传的文件只能读一次。再看了下文档，看到了 Request 的 ParseMultipartForm method 会给 Request.MultipartForm 赋值（其实之前是没看懂），而这个 MultipartForm 就比前面的 MultipartReader 好用些，文件也可以多次读取，新的 [sample](https://github.com/radaiming/dumpling/blob/14c552d99f1f9e1ca7066422929b8a7ee8602e01/_samples/return_hash.go) 也稍微好看了那么一点。

而 test 这边，其实很久没写过测试了，一开始的时候看到 goat 里像[这样](https://github.com/bahlo/goat/blob/4de72452b8cfe7dc1dd9acabf3ca8d4d443dd1ad/middleware_test.go#L8)的 test，感觉这样没什么意义，就大半夜吭哧吭哧写了个[脚本](https://github.com/radaiming/dumpling/blob/16f8b7642d2bde734712373cbdeda34698000456/test_samples.py)去拿自己的 sample 程序当测试用，检查 client 或者 server 输出。等第二天的时候感觉自己[生成 Request ](https://github.com/radaiming/dumpling/commit/16f8b7642d2bde734712373cbdeda34698000456)去测试或者用 [httptest](https://github.com/radaiming/dumpling/commit/2803cc080b647e17a2c900a6eb2408083d6409fc) 也能达到效果，就默默删掉了脚本。

其实写这个框架，更多的时间是在补各种知识，啊。

另外，不知道结尾怎么写，所以拖了俩月才提交。。。