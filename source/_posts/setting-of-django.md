---
title: setting,开发的基本设置
date: 2017-01-17 11:19:35
tags:
- django
- python
---

## setting,开发的基本设置

`ALLOWED_HOSTS = ["*"]`
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME':'moocdai',
        'USER':'django',
        'PASSWORD':'xxxxx',
        'HOST':'10.10.10.3',
    }
}
```
`TIME_ZONE = 'Asia/Shanghai'`

添加app的另外一种方法
```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
##  render,取代原来的template方法
shortcut 是django中一些有用的便捷方法
```python
from django.shortcuts import render
def index(request):
    latest_question_list=Question.objects.order_by('-pub_date')[:5]
    context={

        'latest_question_list':latest_question_list
    }
    return render(request,'polls/index.html',context)
```
## get_object_or_404,连接视图和模型,官方推荐?
```python
def detail(request,question_id):
    question = get_object_or_404(Question,pk=question_id)
    return render(request,"polls/detail.html",{'question':question})
```
## get_list_or_404 上面的兄弟方法,将模型的get()方法变成filter()方法

## 神奇的外键取值
```html
<h1>{{ question.question_text }}</h1>
<ul>
  {% for choice in question.choice_set.all %}
  #这里question是choice的外键,通过question可以取得关联的choice的集合
  <li>{{ choice.choice_text }}</li>
  {% endfor %}
</ul>
```
