---
title: python微信自动化框架wxauto进行消息采集转发
date: 2024-03-07 16:07:37 +0800
tags: 微信自动化 wxauto python
---
## 需求
- 采集大群中含关键词的聊天记录，并进行自定义消息处理「脱敏、添加」，然后转发至指定群聊；同时存储信息至redis key为当天日期的set，判重发送
- 定时任务，每天定时发送公告，公告文件为txt

## 环境
微信版本: **3.9.8.15**

python版本: **3.11.6**

wxauto版本: **3.9.8**

服务器版本: **阿里云 2核4g windows 2012**

## 源码

```python
from wxauto import WeChat
import time
import re
import hashlib
import redis
import datetime
import schedule

# 实例化微信对象
wx = WeChat()

listen_list = [
    '采集群1'
]
# 转发目标群
target_list = [
    #'test',
]

# 文末信息
end_text = "接单请联系群主 📞1234567890"

pool = redis.ConnectionPool(host='127.0.0.1', port=6379, decode_responses=True)


def listen_group_chat():
    for i in listen_list:
        wx.AddListenChat(who=i, savepic=False)  # 添加监听对象并且自动保存新消息图片

    # 持续监听消息
    wait = 10  # 设置10秒查看一次是否有新消息
    while True:
        msgs = wx.GetListenMessage()
        print(f'采集时间：{datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")}')
        for chat in msgs:
            msg = msgs.get(chat)  # 获取消息内容
            print(f'\n----->已获取新的消息,采集源：{chat.who}')
            # 获取每条聊天信息
            for msg_item in msg:
                # 判断是否包含关键词
                msg_content = msg_item[1]
                print(f'消息内容:{msg_item}\n')
                if keyword_match(msg_content):
                    # 判断redis中是否存在
                    if not check_value(msg_content):
                        print("开始自定义消息处理")
                        # 消息脱敏处理
                        cleaned_text = msg_handler(msg_content)
                        print(f"处理结果:{cleaned_text}")
                        # 转发至其他群
                        trans_to_another_group(target_list, cleaned_text)
                        # redis缓存
                        set_cache(msg_content)
                        print("----->消息已缓存,处理结束\n")
                    else:
                        print("----->当日消息已存在， 不作处理\n")
                else:
                    print("----->消息不包含关键词， 不作处理\n")
        schedule.run_pending()
        time.sleep(wait)


# 转发至其他群
def trans_to_another_group(group_names, msg, at=None):
    print(f"即将开始消息转发")
    for group_name in group_names:
        wx.ChatWith(group_name)
        if at:
            wx.AtSomeOne(msg, at)
        else:
            wx.SendMsg(msg)
        print(f"消息已转发至{group_name}")


# 消息处理器
def msg_handler(text):
    # 去除原有手机号
    phone_pattern = "1[3-9]\d{9}"
    cleaned_text = re.sub(phone_pattern, "", text)
    return f'{cleaned_text}\n{end_text}'


# 关键词匹配
def keyword_match(text):
    # 定义要匹配的关键词列表
    keywords = ["key1", "key2", "key3"]
    xiaoqu_keywords = ['area1', 'area2', 'area3']
    # 构建正则表达式
    pattern = r'|'.join(map(re.escape, keywords + xiaoqu_keywords))
    # 进行匹配并输出结果
    matches = re.findall(pattern, text)
    if len(matches) > 0:
        return True
    else:
        return False


# md5加密
def md5_encode(text):
    md = hashlib.md5(text.encode())  # 创建md5对象
    md5pwd = md.hexdigest()  # md5加密
    return md5pwd


# redis set赋值 过期默认24小时
def set_cache(value, timeout=24 * 60 * 60):
    r = redis.Redis(connection_pool=pool)
    key = datetime.datetime.now().strftime("%Y%m%d")
    r.sadd(key, value)


# 判断redis中set值是否存在
def check_value(value):
    key = datetime.datetime.now().strftime("%Y%m%d")
    r = redis.Redis(connection_pool=pool)
    return r.sismember(key, value) != 0


# 广告播报
def notice_broadcast():
    file_path = 'notice1.txt'
    try:
        # 打开文件并读取所有行（包括换行符）
        with open(file_path, 'r', encoding='utf-8') as file:
            content = file.read()
        trans_to_another_group(target_list, content)
    except FileNotFoundError:
        print("未找到该文件")
    except Exception as e:
        print("发生错误: ", str(e))


def schedule_init():
    # 清空任务
    schedule.clear()
    # 创建一个按3秒间隔执行任务
    schedule.every().day.at("06:00").do(notice_broadcast)
    schedule.every().day.at("12:00").do(notice_broadcast)
    schedule.every().day.at("18:00").do(notice_broadcast)
    schedule.every().day.at("21:00").do(notice_broadcast)


if __name__ == '__main__':
    #需要注意 开启定时任务需要在监视循环内 并且初始化有先后顺序
    schedule_init()
    listen_group_chat()

```

## 效果图
![](/img/2024/03/xiaoguotu.png)

## 待优化
- 日志文件输出
- 记录入库
- 公告在线发布

## 常见问题
- 报错 一般是python版本号问题
- 此工具需要保持远程连接，前台运行hook，windows自带工具不可用，本人使用todesk，其他软件自行搜索