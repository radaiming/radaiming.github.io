---
layout: post
title: "升级了一下密码生成器"
categories: security
---

自从几年前 CSDN 被脱裤之后就开始一站一密了，密码生成手段是 hash(salt + site\_name)[0:20] 这样的，site_name 每次敲命令的时候输入，然后 Mac 调 cliclick，Linux 调 xdotool 去模拟键盘打密码。为了防止 salt 被轻易读到，脚本权限改成了 700，这样每次需要生成密码都要 sudo 运行脚步。

但是作为被害妄想，自己当然不会满足于此，时不时会想办法“加固”一下。后来看到 bcrypt 这个计算速度很慢的 hash，就想大概能利用一下，最后想出了这么个逻辑：

~~~~~~~~
encrypted_salt = 'blabla'
decrypt_pw = user_input()
hashed_decrypt_pw = bcrypt(decrypt_pw)
salt = decrypt(hashed_decrypt_pw, encrypted_salt)

print hash(salt + site_name)
~~~~~~~~

salt 是加密的，而且密钥足够长，暴力破解很难；而密钥又是 bcrypt 生成的 hash，因为 bcrypt 慢，所以暴力破解也很难。这么一来，即使脚本明文被读取走也不用太担心。因为 Go 的 bcrypt 库没法自定义 salt（好蛋疼，bcrypt 又不是只能拿来给后端用），所以最后只好用 Python 写了[一个](https://github.com/radaiming/misc/tree/master/python/genpw/genpw.py)。

不过在危险的环境下当然还是会有问题：键盘记录、读取进程内存、添加 LD_PRELOAD 引入其它 so 文件、污染 Python 解释器／库文件甚至编译器，太多手段偷到用户输入的密码。下一步大概要考虑引入 [OTP](https://en.wikipedia.org/wiki/One-time_password) 了。