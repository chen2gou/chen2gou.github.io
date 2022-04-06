---
title: mac上docker mysql启动错误问题排查
date: 2022-04-06 16:25:54 +0800
tags: docker mysql mac
---

## 问题
mbp版本2021款14寸m1 pro芯片，系统Monterey，今天突然发现本地docker中的mysql启动失败，具体报错如下
```
Error invoking remote method 'docker-start-container': Error: (HTTP code 500) server error - error while creating mount source path '/host_mnt/private/var/db/timezone/tz/2021a.3.0/zoneinfo/Asia/Shanghai': mkdir /host_mnt/private/var/db/timezone/tz/2021a.3.0: operation not permitted
```

## 排查
看了下大概是挂载的时间有问题，创建的时候又没权限，查了下本地目录下确实没这个`2021a.3.0`文件夹,但是mysql挂载的是/etc/localtime，映射到具体的时间是`2022a.1.0`,不知道docker抽什么风，重启都不好使.

谷歌搜了一圈看到一个差不多的问题，解决方法是复制一份改名字。尝试了一下，mac有sip完整性保护，系统级别的文件夹都不允许操作，于是`关闭sip`,具体方式是重启后长按开机键，进入系统终端里运行命令`csrutil disable`关闭。

运行如下命令，成功创建文件夹，再运行mysql镜像成功跑起来了。
```
sudo cp -rfp /private/var/db/timezone/tz/2022a.1.0 /private/var/db/timezone/tz/2021a.3.0

```


