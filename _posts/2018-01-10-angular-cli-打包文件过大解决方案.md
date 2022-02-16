---
title: angular-cli 打包文件过大解决方案
date: 2018-01-10 13:53:29
tags:
- angular
---
<img src="https://i.loli.net/2018/01/10/5a55aa9be8884.png" width="40%" height="40%">
### 写好的angular网站首页访问速度很慢，探究一番发现问题所在并解决
<!-- more -->
## ng build生成文件过大


![111.png](https://i.loli.net/2018/01/10/5a55ab90db79f.png)

可以看到5.4M，访问时间达到了12s，一开始以为是什么依赖过大，后来发现不经过压缩生成的文件就是这么大。解决方案：
### 1.配合命令压缩打包

```
 ng build --prod --aot  --build-optimizer
 
```
![xxx.png](https://i.loli.net/2018/01/10/5a55ac68b7988.png)


### 2.再配合nginx开启gzip压缩

![xxx.png](https://i.loli.net/2018/01/10/5a55acbd0e5b7.png)

* 可以看到最后文件只有400kb大小，访问速度达到最快

**最后感谢[pcrobot](http://pcrobot.me)道友一同解决问题**
