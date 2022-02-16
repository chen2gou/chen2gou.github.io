---
title: onos&mininet基于SDN模拟ip+光安装指北
date: 2019-05-20 10:39:58
tags: onos mininet
---
> onos mininet 真挺坑的，国内研究人很少，相关文档基本靠墙外，记录下过程吧

<!--more-->
### 1.onos安装
> 基于1.9.0安装onos
1. onos github下载源码，切换1.9.0分支
2. 设置环境变量，修改 `tools/dev/bash_profile` 中各项配置，系统变量`.profile`中加入如下

```
source ~/documents/work/onos/tools/dev/bash_profile #刚刚下载onos目录
export ONOS_APPS="drivers,drivers.optical,openflow,proxyarp,optical,onos-jsptpd-sdn" #onos-jsptpd-sdn为另外开发app
export PATH="$PATH:$HOME/bin"
```
3. 项目目录中运行 `ok` ,此时会下载buck并打包运行项目，会比较慢，一定要挂梯子
4. `op`打包命令，`ok clean debug`会开启远程端口，配合idea可以实现调试


### 2.mininet安装
> onos 官网教程

```
cd
git clone git://github.com/mininet/mininet
cd mininet
wget 'https://wiki.onosproject.org/download/attachments/4164175/multi_controller.patch?version=1&modificationDate=1443649770762&api=v2' -O multi_controller.patch
git apply multi_controller.patch
sudo ./util/install.sh -3fnv
# role back CPqD to a version known to work
cd
cd ~/ofsoftswitch13/
make clean
git reset --hard 8d3df820f7487f541b3f5862081a939aad76d8b5
sudo make install
cd
sudo mn --test pingall # this should work
```

### 3.linc-oe安装
> 模拟光网络

```
cd
git clone git://github.com/mininet/mininet
cd mininet
wget 'https://wiki.onosproject.org/download/attachments/4164175/multi_controller.patch?version=1&modificationDate=1443649770762&api=v2' -O multi_controller.patch
git apply multi_controller.patch
sudo ./util/install.sh -3fnv
# role back CPqD to a version known to work
cd
cd ~/ofsoftswitch13/
make clean
git reset --hard 8d3df820f7487f541b3f5862081a939aad76d8b5
sudo make install
cd
sudo mn --test pingall # this should work
```

### 4.新建拓扑推送到onos

> mininet安装补丁后可以推送多个onos主机，实现集群
```
sudo -E python onos/tools/test/topos/opticalTestBig.py $OC1 $OC2 $OC3
```

### 5.实现rest接口操作mininet


github上找到一个项目 
[mininetrest](https://github.com/chen2gou/mininetRest)，二次开发了下，实现操作link，启动mininet的时候会启动一个web服务器监听请求操作mininet




参考文档

[使用buck安装ONOS](https://blog.csdn.net/weixin_33767813/article/details/87511872)

[onos ip+光环境配置](https://wiki.onosproject.org/display/ONOS/The+Packet+Optical+Dev+Environment)