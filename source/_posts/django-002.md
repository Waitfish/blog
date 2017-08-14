---
title: django migrate 出错的处理
date: 2017-01-13 14:31:01
tags:
- django
- python
---
# django migrate 出错的处理

如果不小心,数据库的一些逻辑没处理好,就容易出现migrate给你报一大堆错误.最暴力的解决办法就是删除migrate文件夹下面的所有文件,然后新建一个__init__.py 文件
然后重新 makemigrationgs  migrate
