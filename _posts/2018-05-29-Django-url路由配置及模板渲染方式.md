## `urls.py`路由用法

#### 知识点

1. `url`基本概念
2. `url`格式
3. `urls.py`的作用
4. `url`解析过程
5. `include`的作用
6. `kwargs`的作用
7. `name`的作用

#### URL概念

```
URL（Uniform Resoure Locator）统一资源定位符是对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址。互联网上的每个文件都有一个唯一的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。
```

#### `URL`格式

```
http://127.0.0.1:8000/hello/
URL解释：
schema://host[:port#]/path/.../[?query-string][#anchor]
schema：指定使用的协议(例如：http, https, ftp)
host：Http服务器的IP地址或者域名
port：端口号，http默认是80端口
path：访问资源的路径
query-string：发送给http服务器的数据
anchor：锚点
```

#### `url.py`的作用

```
URL配置（URLconf）就像是Django所支撑网站的目录。它的本质是URL模式以及要为该URL模式调用的视图函数之间的映射表。以这样的方式告诉Django，对于那个URL调用那段代码。url的加载就是从配置文件中开始。
```

#### `url`解析过程

```
Django会从urlpatterns里顺次读取元素，每个元素是方法url调用后返回的结果。
		django传给url路由要处理的地址，该地址是被去掉主机地址及之后的一个“/”的剩余部分：
		例如： http://127.0.0.1:8000/hello/
		经处理后的剩余部分就是：
		hello/
		然后把该剩余部分与url方法的第一个参数进行正则匹配，如何匹配成功，则执行url的第二个参数指定的方法（该方法一般放在views.py中，主要功能的实现具体业务逻辑）。
```

#### `url`例子

```python
from django.conf.urls import url
from basketball.views import (
    index,
    test,
    new,
    home,
    people,
    traditional,
)

urlpatterns = [
    url(r"^$",index),
    url(r"^home/(.+)/$",home),
    url(r"^traditional/$",traditional),
    url(r"^people/(?P<name>\w+)/(?P<age>\w+)/$",people),
    url(r"^test/$",test,name='bask_test'),
    url(r"^new/(?P<name>\w+)/(?P<age>\w+)/$",new,name='bask_new'),
]

```

```python
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.
def index(request):
    name = request.GET.get('name',"nobody")
    lis = ['a','b','c','d']
    return render(request,"basketball.html",{
        'name':name,
        'lis':lis,
    })
def home(request,name):
    return HttpResponse("这是{}的家".format(name))
def people(request,name,age):
    return HttpResponse("这是:{}今年{}".format(name,age))
def traditional(request):
    name = request.GET.get('name',None)
    age = request.GET.get('age',None)
    return HttpResponse("名字:{};年龄:{}".format(name,age))

def test(request):
    return HttpResponse("<h1>这是测试页面</h1>")

def new(request,name,age):
    # print(name)
    # print(age)
    return HttpResponse("<h1>姓名:{}<br />年龄:{}</h1>".format(
        name,age
    ))


```

参数捕获：

1、捕获位置参数（可变参数）：在url函数中，第一个正则表达式使用（）括号进行捕获参数.

2、捕获关键字参数：在url函数中，第一个正则表达式使用`（?P<keyword>）`进行捕获。

注意事项：

参数类型是字符串类型，所以，如果使用数字类型，需要使用int函数转换成int类型。

#### `include`的作用

```
方便项目管理：
	一个project有一个总的urls.py，各个app也可以自己建立自己的urls.py，不过都需要使用include()函数在project的urls.py文件进行注册。
```

```python
#hello_django/urls.py     主url文件
from django.conf.urls import include,url
from django.contrib import admin
from .import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^books/',include('books.urls')),
]
```

```python
#books/urls.py    APP books里面的URL文件
from django.conf.urls import url
from . import views

urlpatterns =[
    url(r'^$',views.index),
    url(r'article/$', views.article),
```

```python
#book/view.py           APP books里面的view文件
from django.http import HttpResponse
def index(request):
    return HttpResponse('这是index页面')
def article(request):
    return HttpResponse('这是article首页')
```

```
Include注意事项：
到达django来的时候，已经有域名和反斜杠了，拿本机地址为例，已经有http://127.0.0.1/这个样子了，所以主url中都不需要以/开头。
子url在匹配的时候，其实就是字符串的匹配，一层套一层的。

主Url匹配，开始的地方不需要加反斜杠。
原因是：因为django已经给域名后面加了一个正斜杠，所以不需要再加，否则将匹配不到正确的URL。
主url后面要加正斜杠。
app的url，前面不要加正斜杠。
主url后面不要加$符号，
子app的url，后面要加$符号。
```

#### `kwargs`的作用

```
传递一个Python 字典作为额外的参数传递给视图函数。django.conf.urls.url() 函数可以接收一个可选的第三个参数，它是一个字典，表示想要传递给视图函数的额外关键字参数。
```

```python
#hello_django/urls.py     主url文件
from django.conf.urls import include,url
from django.contrib import admin
from .import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^books/',include('books.urls'),{'switch':'true'}),
]
```

```python
# book/view.py      APP books里面的view文件
from django.http import HttpResponse
import datetime
def index(request,**args):
    if args.get('switch') == 'true':
        print(datetime.datetime.now())
    return HttpResponse('<h1>这是index页面</h1>')
```

```
URL传递额外参数：
url函数有一个参数叫做kwargs，这个参数可以传递额外的参数到views中，并且必须为字典类型。这在使用include的时候，需要统一给下面的url一些参数的时候，显得尤其有用。
```

#### `name`的作用

```
给一个匹配的url地址取名字
一般用于模板
也可以使用reverse进行页面重定向
```
```python
from django.conf.urls import url
from basketball.views import (
    test,
    new,
    redirect_test,
)

urlpatterns = [
    url(r"^test/$",test,name='bask_test'),
    url(r"^redirectToTest/$",redirect_test),
]

```

```python
from django.shortcuts import render,redirect,reverse
from django.http import HttpResponse
# Create your views here.
def test(request):
    return HttpResponse("<h1>这是测试页面</h1>")
def redirect_test(request):
    """重定向到'back_test'这个路由的页面"""
    return redirect(reverse('bask_test'))

```

```
url有一个name的参数，name参数可以给这个url取一个合适的名字。通过给url取名字，以后在view或者模板中使用这个URL，就只需要通过这个名字就可以了。这样做的原因是防止url的规则更改，会导致其他地方用了这个url的地方都需要更改，但是如果取名字了，就不要做任何改动了。
```

# 模板渲染方式

1. 模板路径
2. 渲染方式

setting.py中模板路径配置:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

1.直接将`html`字符串硬编码`HttpResponse`中

```python
def index_1(request):
    return HttpResponse('<h1>Hello Django World!</h1>')
```

2.`django.template.loader` 定义了函数以加载模板。

`get_template`（*template_name*，*using = None*）	

该函数使用给定的名称加载模板并返回一个`Template` 对象.`get_template()`尝试获取每个模板直到有一个成功满足。

注意获取模板的顺序:先主项目的templates里面-->app的templates里面

```python
def template_render(request):
    """04加入模板html"""
    t = get_template("hello_django.html")
    html = t.render()  #render()括号里面可遗传字典参数context
    return HttpResponse(html)
```

3.使用render进行渲染。

```python
def simple_render(request):
    """05加入模板html"""

    return render(request,"hello_django.html")
#render(request, template_name, context=None, content_type=None, status=None, using=None)
```

#### 
