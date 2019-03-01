### 1 无参数情况

配置URL及其视图如下:

```python
urlpatterns = [
    url(r"^$",index),
    ]
 def index(request):
 	return HttpResponse("我是index")
```

访问http://127.0.0.1:8000/index，输出结果为“我是index”

### 2 传递一个参数

配置URL及其视图如下,URL中通过正则指定一个参数:

```python
urlpatterns = [
  url(r"^home/(.+)/$",home),
]

def home(request,name):
    return HttpResponse("这是{}的家".format(name))
```

访问http://39.105.77.167:8000/basketball/home/xiaoming/

输出结果:这是xiaoming的家

### 3 传递多个参数

参照第二种情况,以传递两个参数为例,

配置URL及其视图如下,URL通过正则

指定两个参数:

```python
url(r"^home/(.+)/$",home),
    url(r"^people/(?P<name>\w+)/(?P<age>\w+)/$",people),
]

def people(request,name,age):
    return HttpResponse("这是:{}今年{}".format(name,age))
```

访问http://39.105.77.167:8000/basketball/people/xiaofang/21/

输出结果:这是:xiaofang今年21

### 4 通过传统的"?"传递参数

例如,http://39.105.77.167:8000/basketball/traditional/?name=xiaojun&age=18,

url中"?"之后表示传递的参数,这里传递了name和age两个参数。

通过这样的方式传递参数,就避免了正则匹配错误

或者匹配不到的情况。在Django中,此类参数的解析是通过request.GET.get或者request.POST.get

方法获取到的。

配置URL及其视图如下:

```python
urlpatterns = [
 url(r"^traditional/$",traditional),   ]
 
 def traditional(request):
    name = request.GET.get('name',None)
    age = request.GET.get('age',None)
    return HttpResponse("名字:{};年龄:{}".format(name,age))
```

输出结果:名字:xiaojun;年龄:18

扩展

```
1 如果使用urls传统?方式传参，
  在html页面就不使用{% url %}模板，来传参
2 如果想使用{% url %}模板来实现传参，那么
  urls传参方式一般都是正则方式来传参，
  比如（分页，文章详情）
3 实际工作中，如果使用urls传统？方式传参，
  前端也页面传递大量参数，一般都是form表单
  方式或者ajax，不会使用，{% ur l%}这样的模板标签
  去传大量数据
```





