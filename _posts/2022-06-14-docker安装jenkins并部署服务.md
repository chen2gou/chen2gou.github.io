---
title: docker安装jenkins并部署服务
date: 2022-06-14 08:29:49 +0800
tags: docker jenkins
---

## 安装
1、新建文件夹jenkins，新建docker-compose.yml文件，默认已安装docker-compose，拷贝如下配置。

``` yaml
version: '3.1'
services:
  jenkins:
    image: jenkins/jenkins:lts
    restart: always
    hostname: jenkins
    container_name: jenkins
    privileged: true  # 特权模式
    user: root
    ports:
    - 7777:8080 #web端口映射
    - 50000:50000
    environment:
      TZ: Asia/Shanghai
      JENKINS_JAVA_OPTIONS: -Djava.awt.headless=true -Xms512m -Xmx1024m -XX:MaxNewSize=512m -XX:MaxPermSize=512m #限制内存大小，小内存哭了
    volumes:
    - /root/env/Jenkins/data:/var/jenkins_home   # jenkins数据目录
    - /root/env/apache-maven-3.5.4:/usr/local/maven # maven映射

```
2、启动jenkins  执行`dokcer-compose up -d`
> 如果修改过配置文件，重新执行此命令会重建容器，除了挂载卷，修改内容都会消失。
 {: .prompt-danger } 
 
3、访问上面映射的端口 localhost:7777,密码可以查看docker logs -f jenkins。
## 配置
1、安装插件gitee、Publish Over SSH，配置maven地址、gitee账号。
> 此处注意maven文件需要配置成如下，在maven根目录新建ck文件夹，同时修改包下载路径，这样包会下载到映射的宿主机文件夹中，便于后期管理
 {: .prompt-info }

  ``` 
  <localRepository>${maven.home}/ck</localRepository> 
  ```
 
2、配置ssh访问，A要访问B主机(本案A为docker jenkins，B为宿主机)，需要A生成ssh凭证，并把公钥传给B，由于jenkins版本原因，生成ssh凭证代码为 `ssh-keygen -m PEM -t rsa -b 4096`,再通过`ssh-copy-id 192.168.66.103`把公钥传输到B。在配置页面进行配置后，点击Test Configuration后显示success即完成Publish Over SSH配置。

## 部署
这里选用是自由风格项目，拉取源代码后进行maven编译，编译完成后进行推送jar包到远程主机，本案为宿主机。然后执行项目下部署脚本，需要注意杀进程可能把jenkins部署进程杀掉，需要在脚本中加入如下配置`export BUILD_ID=dontKillMe
source /etc/profile`

``` shell
#!/bin/bash

#jar包文件夹路径及名称
APP_NAME=/root/java/haoayi/jbhay.jar

#日志文件路径及名称
LOG_FILE=log/application_log.log

#重启命令
pid=`ps -ef | grep $APP_NAME | grep -v grep |awk '{print $2}'`
if [ $pid ]
then
kill -9 $pid
fi

# 启动jar包，指向日志文件，2>&1 & 表示打开或指向同一个日志文件
export BUILD_ID=dontKillMe
source /etc/profile
nohup  java -jar $APP_NAME --spring.profiles.active=pro >/dev/null 2>&1 &
```

