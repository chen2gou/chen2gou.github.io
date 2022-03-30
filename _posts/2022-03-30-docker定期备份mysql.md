---
title: docker定期备份mysql
date: 2022-03-30 08:24:21 +0800
tags: 运维 linux mysql
---

## 原因
这不是前面迁移了两个系统吗。所有运维都得自己来，数据无价，为了防止猪队友操作，所以准备定期备份mysql数据。就在备份完的之后n天，新入职的张三就把测试库导到生产库了，得，我就是预言家。

## mysql设置
首先，mysql备份用到`mysqldump`命令，需要在mysql配置文件中增加如下配置，否则需要在命令行中加上用户密码。
```
[mysqldump]
user=xxx
password=xxx
```
## 脚本
> 注意点：以下所有操作都为root权限。
{: .prompt-warning }

```
vim mysqlback.sh
```
复制粘贴如下脚本
```
# 保留10天数据，
# mysql-service为安装docker中mysql服务名称
docker exec -i mysql-service bash<<'EOF'
# 判断目录是不是已经存在，如果不存在则创建
if [ ! -d "/backups/mysql" ]; then
  mkdir -p /backups/mysql
fi
# xxx 为数据库的名称
mysqldump xxx > /backups/mysql/backups_$(date +%Y%m%d).sql
#删除超过10天的数据
rm -f /backups/mysql/backups_$(date -d -10day +%Y%m%d).sql
exit
EOF

# 宿主机操作：判断目录是不是已经存在，如果不存在则创建
if [ ! -d "/home/admin/backups/mysql" ]; then
  mkdir -p /home/admin/backups/mysql
fi
# 将docker中的备份的数据拷贝到宿主机上。
docker cp mysql-service:/backups/mysql/backups_$(date +%Y%m%d).sql /home/admin/backups/mysql
#删除超过10天的数据
rm -f /home/admin/backups/mysql/backups_$(date -d -10day +%Y%m%d).sql
```
## 定时任务
打开定时任务配置
```
crontab -e
```
增加如下命令
```
59 23 * * * sh /home/admin/env/mysqlbak.sh
```