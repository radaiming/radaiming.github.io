---
layout: post
title: "FFmpeg VA-API 硬解/硬编"
categories: TLDR
---

以前还年轻的时候 H.264 还没统治世界，下的片儿什么格式都有，要想在计算力孱弱的移动设备上播放经常只能靠硬解，于是就学着转码，一开始用的是 Mplayer 自带的 MEncoder，但编出来的视频经常是花的，后来在用了 FFmpeg 一段时间后发现这货实在太靠谱了，这种强烈对比下让我感叹 FFmpeg 怎么这么赞。不过当年电脑上只有硬解，硬编这边既没有 NVENC，更没有 VA-API，只能烧 CPU 慢慢编，通宵编。

后来工作后就基本没这种需求了，直到公司试图在产品里使用硬编码，我才知道 Intel 有个拿来卖钱的 Media SDK，需要 build 修改过的 kernel（其实主要就改了 i915），然后调用他们提供的库。因为是商业产品，我又不会写 C，也不能拿人家商业授权来私用，所以也没啥想法。

然后前段时间在 Phoronix 上看到一条 [FFmpeg Lands VA-API H.264 / H.265 / MJPEG Encoder Support](https://www.phoronix.com/scan.php?page=news_item&px=FFmpeg-VA-API-Encoder)，心里一阵卧槽，Intel 不是靠这个卖钱的么。昨晚想起这个就想试一下，结果因为好几个软件都依赖低版本的 FFmpeg，还没法直接升到 9999 版本。不过只是为了尝试下于是就暂时 break 掉依赖，直接 emerge --nodep 给装上去，然后根据 [mail list](https://ffmpeg.org/pipermail/ffmpeg-user/2016-May/032153.html) 里的回答试了下，编码速度碉堡，如果再加上 VA-API 硬解，那 CPU 占用就低到只吃掉了半个核了：

~~~
ffmpeg -hwaccel vaapi -vaapi_device /dev/dri/renderD128 -i IMG_0878.MOV -vf 'format=nv12,hwupload' -c:v h264_vaapi test.mkv
~~~

测试用的是自己拍的 4k 视频，从 intel\_gpu_top 命令来看，GPU 只被吃掉了一半，不清楚瓶颈在哪。而 speed 的话达到了约 0.8x，就是每秒能编码出 0.8 秒的视频，要知道在没有硬解和硬编的情况下 speed 也就 0.06x 这样。不过编码出来的视频在快进的时候会花，不知道是 FFmpeg 还是 libva 的锅。

FFmpeg 能工作了，但依赖它的包可能就挂了，所以还是得先把它回退。而 FFmpeg 官方推荐的 [static build](http://johnvansickle.com/ffmpeg/) 直接报错找不到 libva 相关的库，看来是 build 的时候并未把 libva 一起静态编进去。每当碰到这种问题就感叹静态链接大法好。最后只能自己想办法写个编静态版本的 ebuild。要静态编译成一个 binary，依赖的库也都得静态编译才行，刚改了一会发现 libva 的 ebuild 并没有提供 static-libs 的 USE，放弃。

不过在尝试启用 FFmpeg 自己的静态编译选项后发现，编出来的虽然还是动态链接的，但 FFmpeg binary 却大了很多，原来是把自身的库给合进了 binary 里，于是现在所依赖的只有别的包的库文件了，这种情况下可以有任意多个 FFmpeg 版本共存了，最后的 ebuild 我也只装了 ffmpeg 和 ffprobe 两个 binary。

唉现在自己的文字写得是越来越水了。