# Celery简介

Celery 是一个 基于python开发的分布式异步消息任务队列，通过它可以轻松的实现任务的异步处理

# Celery的相关概念

## task

​	需要执行的任务

## worker

​	负责干活儿的小弟

## broker

​	任务队列（worker拿任务的地方）

## backend

​	干完活儿 结果存放的位置

# Celery基本工作流程

   ![celery工作流程图](http://p7bj6aatj.bkt.clouddn.com/celery%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

# Celery的安装与使用

## 安装

~~~
sudo pip install celery
sudo pip install celery-with-redis
sudo pip install django-celery
sudo apt install redis-server
~~~

## 配置

### settings.py文件

~~~python
ALLOWED_HOSTS = ['*']
INSTALLED_APPS = [
	  ...
	  'djcelery',
	  ‘自己的APP’
	]
#---celery.conf----
import djcelery
djcelery.setup_loader()
BROKER_URL="redis://127.0.0.1:6379/1"	#这里的1代表使用redis的哪个库
CELERY_CONCURRENCY=2    #(设置worker的并发数量)
CELERY_RESULT_BACKEND = "redis://127.0.0.1:6379/3"#---celery.conf----
import djcelery
djcelery.setup_loader()
BROKER_URL="redis://127.0.0.1:6379/1"
CELERY_CONCURRENCY=2    #(设置worker的并发数量)
CELERY_RESULT_BACKEND = "redis://127.0.0.1:6379/3"


#计划任务

CELERYBEAT_SCHEDULE={
    'every-two-seconds-run-create_data':{
        'task':'t9.tasks.create_data',
        'schedule':timedelta(seconds=2),
        'args':()
    },
    'every-week-three-run-get_data_with_param':{
        'task':'t9.tasks.get_data_with_param',
        'schedule':crontab(day_of_week="3,4"),
        'args':()
    }
}
~~~



```python
#----day09\celery.py-----

from __future__ import absolute_import
from celery import Celery
from django.conf import settings
import os

#设置系统的环境配置用的是django的
os.environ.setdefault('DJANGO_SETTING_MODULE','day09.settings')

#实例化celery
app = Celery("mycelery")
app.conf.timezone="Asia/Shanghai"
#指定celery的配置来源
app.config_from_object("django.conf:settings")
#让celery 自动发现我们的task任务
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
#一定要注意，tasks.py文件不要写错,要不然找不到
#你需要在app目录下  新建一个叫tasks.py的文件
```



### settings.py同级目录下的__init__.py加入

~~~python
from __future__ import absolute_import
#绝对引用，位置必须在第一行
#python2需要这样写
from __future__ import absolute_import, unicode_literals
from .celery import app as celery_app
~~~

## 使用

### 	1、在需要使用异步任务的APP目录下新建tasks.py

~~~python
#---------t9\tasks.py-----------------


import time
from celery import task
from django.http import HttpResponse
import random

from .models import MyData
#文件写入的基本都是task函数

@task
def get_data():
    #假装在查数据
    time.sleep(3)
    # return HttpResponse('我正在running  ok')
    print('我正在running  ok')

@task
def get_data_with_param(loop):
    for i in range(loop):
        time.sleep(2)
        print("我被第{}次执行了".format(i))


@task
def create_data():
    num = random.randrange(100)
    MyData.objects.create(data="{}".format(num))
~~~

### 2、views.py内的调用

~~~python
#-------t9\views.py---------------------


#任务函数名.delay(参数，，，，)


from django.http import HttpResponse
from django.shortcuts import render

from t9.models import MyData
from .tasks import get_data,get_data_with_param,create_data
from .my_singals import action
# Create your views here.

def index(request):
    # get_data()
    # loop = 3
    # get_data_with_param.delay(loop)
    create_data()
    return HttpResponse("我是辅助")


def test_singal(request):
    # obj = MyData()
    # obj.data = "hehe"
    # obj.save()
    action.send(sender="dada",data="hahaha")
    return HttpResponse("ok")
~~~

### 3、python    manage.py     migrate 建表

### 4、启动worker python   manage.py   celery   worker   --loglevel=info

​	**** (或者celery -A 你的工程名 worker -l info)

​	**注意修改tasks.py的时候要重启celery**

### 5、注意也要启动你的django项目

​	`python manage.py runserver 0:8000`

## 定时任务

~~~python
在settings.py文件添加
CELERYBEAT_SCHEDULE={
    'every-two-seconds-run-create_data':{
        'task':'t9.tasks.create_data',
        'schedule':timedelta(seconds=2),
        'args':()
    },
    'every-week-three-run-get_data_with_param':{
        'task':'t9.tasks.get_data_with_param',
        'schedule':crontab(day_of_week="3,4"),
        'args':()
    }
}

~~~

**启动： celery -A 你的工程名称 beat -l info（或者python manage.py celery beat --loglevel=info）**

### 计划任务时间

~~~
from celery.schedules import crontab
crontab(minute=u'00', hour=u'11',day_of_week='mon,tue,wed,thu,sun')
~~~





## 坑:

我们启动定时任务的时候

首先要开启worker

如果只开启定时任务，定时任务会被放入任务队列；

此时如果没有开启worker,那么任务是没有被执行的；

此时如果开启了worker,那么任务队列是有任务的，就去执行；（也就是说，就算你定时任务没开，无所谓，只要任务队列只要有任务，我就去执行）