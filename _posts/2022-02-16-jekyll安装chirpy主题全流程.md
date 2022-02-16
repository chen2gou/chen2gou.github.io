---
title: jekyll安装chirpy主题全流程
date: 2022-02-16 15:20:20
tags: 博客
---

## 安装jekyll
此步骤略过，网上较多，全程需要科学上网，需要注意的是mac自带的ruby会有问题，升级下rvm安装高版本ruby可以正常使用。此处使用的版本是`ruby 2.6.6p146 (2020-03-31 revision 67876) [x86_64-darwin21]`，吐槽下ruby的环境安装真是一言难尽，劝退一大波人。

## 选择主题
此处直接fork[**Chirpy Starter**](https://github.com/cotes2020/chirpy-starter/generate)，拿回来改改就好使了，主题风格极简深得我心。

![首页](/img/2022/chripy_home.png)

> 注意点：项目fork后需要修改Gemfile的源地址，`source "https://gems.ruby-china.com"`，这时候运行`bundle`会安装相关依赖,运行`jekyll s`即可看到初始化首页。
{: .prompt-warning }


## 博客配置

博客配置位于_config.yml，修改中文`lang: zh-CN`以及一些基础信息；默认不编译时间大于当前时间的文章，可增加`future: true`去除限制。

重点说下评论配置，本人选用的是giscus，利用github自带的discussion讨论区功能来进行评论管理，使用giscus有三个先决条件，可以参考[**giscus-cn**](https://giscus.app/zh-CN)。

按照配置完成后，我们博客配置文件中`comments.active`后填入`giscus`，giscus后repo填的是自己准备当做评论功能的repo，格式`用户名/项目名`，repo_id是项目id，category是discussion分类，category_id是分类id，mapping是映射方式可以选择很多种，id不清楚可以打开上述网址后，输入仓库等一系列配置后，在下方的script脚本中看到对应数据,完成后即可在文章下方看到评论区。

## 安装部署
区别于hexo需要编译后再上传文件，github支持jekyll自动编译部署，上传代码后github自动调用workflow中脚本生成一个`gh_pages`分支,此时在setting-pages里面选择分支设置好域名即可，相关教程可以搜索`pages部署博客`,enjoy it！