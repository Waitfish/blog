---
title: django 学习(1)
date: 2017-01-13 10:16:30
tags:
- python
- django
---
# 开始

## 数据库配置
- setting.py 文件中设置
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME':'moocdai',
        'USER':'django',
        'PASSWORD':'xxxxx',
        'HOST':'10.10.10.3',
    }
```
- 在 ubuntu 中安装mysql支持
```
sudo apt-get install libmysqlclient-dev
pip install MySQL-python
```

## 让系统使用我们重载的 userprofile 类
setting.py 中设置
```
from users.models import UserProfile
AUTH_USER_MODEL="users.UserProfile"
```

## 在modeles中使用imagefield的话,就要安装pillow
`pip install pillow`
## 如果要用xadmin,现在要用django1.9需要再安装 django-reversion

## django migrate 出错的处理

如果不小心,数据库的一些逻辑没处理好,就容易出现migrate给你报一大堆错误.最暴力的解决办法就是删除migrate文件夹下面的所有文件,然后新建一个__init__.py 文件
然后重新 makemigrationgs  migrate
