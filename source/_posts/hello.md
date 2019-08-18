---
title: APScheduler(Python)定时框架
date: 2019-08-18 11:05:02
tags:
---

##### 前言

想写一个每天在固定时间爬取必应壁纸并对Windows系统壁纸进行更换的小程序部署到服务器，拜读了biglao的[博客](https://lz5z.com/Python%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F/)，站在巨人的肩膀人学习一下[APScheduler](https://apscheduler.readthedocs.io/en/latest/index.html)。做一个Programmer的乐趣就是能用code做一些快乐的事情吧。理想的成果是将代码放在服务器上跑，定时爬取到图片后传递回本机，然后再设置本地壁纸。

##### 间隔性任务

``` python
# -*- coding: utf-8 -*-
# @Time    : 2019/6/24 7:05
# @Author  : 2simple
# @Site    : 时间间隔型任务
# @File    : demo-1.py
# @Software: PyCharm

from datetime import datetime
import os
from apscheduler.schedulers.blocking import BlockingScheduler


def tick():
    print('Tick! The time is: %s' % datetime.now())


if __name__ == '__main__':
    scheduler = BlockingScheduler()
    scheduler.add_job(tick, 'interval', seconds=3)
    # 若当前OS为NT则按Ctrl+Break键退出，否则按Ctrl+C,部分不带小键盘的笔记本电脑上没有break键
    print('Press Ctrl+{0} to exit'.format('Break' if os.name == 'nt' else 'C    '))
    try:
        scheduler.start()
    except (KeyboardInterrupt, SystemExit):
        pass
```

参数

- weeks (int) – number of weeks to wait
- days (int) – number of days to wait
- hours (int) – number of hours to wait
- minutes (int) – number of minutes to wait
- seconds (int) – number of seconds to wait
- start_date (datetime|str) – starting point for the interval calculation
- end_date (datetime|str) – latest possible date/time to trigger on
- timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations

##### 定时性任务

```python
# -*- coding: utf-8 -*-
# @Time    : 2019/6/24 7:42
# @Author  : 2simple
# @Site    : 定时性任务
# @File    : demo-2.py
# @Software: PyCharm


from datetime import datetime
import os
from apscheduler.schedulers.blocking import BlockingScheduler


def tick():
    print('Tick! The time is: %s' % datetime.now())


if __name__ == '__main__':
    scheduler = BlockingScheduler()
    # 每天8:48执行任务
    scheduler.add_job(tick, 'cron', hour=8, minute=48)
    print('OS is {0} ,Press Ctrl+{1} to exit'.format(os.name, 'Break' if os.name == 'nt' else 'C    '))

    try:
        scheduler.start()
    except (KeyboardInterrupt, SystemExit):
        pass
```

参数：

- year (int|str) – 4-digit year
- month (int|str) – month (1-12)
- day (int|str) – day of the (1-31)
- week (int|str) – ISO week (1-53)
- day_of_week (int|str) – number or name of weekday (0-6 or mon,tue,wed,thu,fri,sat,sun)
- hour (int|str) – hour (0-23)
- minute (int|str) – minute (0-59)
- second (int|str) – second (0-59)
- start_date (datetime|str) – earliest possible date/time to trigger on (inclusive)
- end_date (datetime|str) – latest possible date/time to trigger on (inclusive)
- timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations (defaults to scheduler timezone)

##### 应用

```python 
import requests
from scrapy import Selector
from apscheduler.schedulers.blocking import BlockingScheduler
from datetime import datetime
import time
import os


def downloadImages(photoUrl, photoName):
    root = 'C://Users/Mr.Su/Desktop/'
    path = root + photoName + '.jpg'
    try:
        # 如果路径不存在，则创建
        if not os.path.exists(root):
            os.mkdir(root)
        if not os.path.exists(path):
            r = requests.get(photoUrl)
            # 将爬取的二进制信息保存为文件(图片)
            with open(path, 'wb') as f:
                f.write(r.content)
                f.close()
                print(time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(time.time())) + photoName + '照片保存成功')
        else:
            print("照片已存在.....")
    except:
        print("下载或保存环节异常")


def getURL(URL, NAME):
    res = requests.get(URL).content
    selector = Selector(text=res)
    href = selector.xpath('//*[@id="bgLink"]/@href').extract()[0]
    photoUrl = URL + href
    downloadImages(photoUrl, NAME)


def Entrance():
    URL = 'https://cn.bing.com/'
    NAME = '{0}月{1}日'.format(datetime.now().month, datetime.now().day)
    # print('网站是{0}，名字是{1}'.format(URL, NAME))
    getURL(URL, NAME)


if __name__ == '__main__':
    scheduler = BlockingScheduler()
    scheduler.add_job(Entrance, 'cron', hour=9, minute=42)
    print('OS is {0} ,Press Ctrl+{1} to exit'.format(os.name, 'Break' if os.name == 'nt' else 'C    '))
    try:
        scheduler.start()
    except (KeyboardInterrupt, SystemExit):
        pass
```

