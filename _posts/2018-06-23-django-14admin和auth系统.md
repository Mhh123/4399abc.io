## 管理站点

#### 创建一个管理员用户

`python manage.py createsuperuser，按提示输入用户名、邮箱、密码`

启动服务器，通过“127.0.0.1:8000/admin”访问，输入上面创建的用户名、密码完成登录

进入管理站点，默认可以对groups、users进行管理

#### 管理界面本地化

编辑`settings.py`文件，设置编码、时区

```
LANGUAGE_CODE = 'zh-Hans'
TIME_ZONE = 'Asia/Shanghai'
```

#### 向`admin`注册模型

```
#----------admin.py---------
from django.contrib import admin
# Register your models here.
from .models import Department,Student,Course

admin.site.register(Department)
admin.site.register(Student)
admin.site.register(Course)
```

刷新管理页面，可以对数据表中数据进行增删改查操作

#### 自定义管理页面

`Django`提供了`admin.ModelAdmin`类

通过定义`ModelAdmin`的子类，来定义模型在`Admin`界面的显示方式

###### 列表页属性

`list_display`：显示字段，可以点击列头进行排序

`list_filter`：过滤字段，过滤框会出现在右侧

`search_fields`：搜索字段，搜索框会出现在上侧

`list_per_page`：分页，分页框会出现在下侧

###### 添加、修改页属性

`fields`：属性的先后顺序

`fieldsets`：属性分组

注意：上面两个属性，二者选一。

###### 例子

```
#-----models.py------
class Department(models.Model):
    d_id = models.AutoField(primary_key=True)
    d_name = models.CharField(max_length=30)
    def __str__(self):
        return 'Department<d_id=%s,d_name=%s>'%(
            self.d_id,self.d_name
        )

class Student(models.Model):
    s_id = models.AutoField(primary_key=True)
    s_name = models.CharField(max_length=30)
    department = models.ForeignKey('Department')
    course = models.ManyToManyField('Course')
    def __str__(self):
        return 'Student<s_id=%s,s_name=%s>'%(
            self.s_id,self.s_name
        )


class Course(models.Model):
    c_id = models.AutoField(primary_key=True)
    c_name = models.CharField(max_length=30)
    def __str__(self):
        return 'Course<c_id=%s,c_name=%s>'%(
            self.c_id,self.c_name
        )
```

```
# -----------admin.py--------
from django.contrib import admin

# Register your models here.
from .models import Department,Student,Course

class DepartmentAdimin(admin.ModelAdmin):
    list_display = ['d_id','d_name']
    list_display_links = ['d_id','d_name']
    list_filter = ['d_id']
    search_fields = ['d_name']

class StudentAdimin(admin.ModelAdmin):
    list_display = ['s_id','s_name']
    list_display_links = ['s_id','s_name']
    # fields = ['s_name','course','department']
    fieldsets = [
        ('一组',{'fields':['s_name']}),
        ('二组',{'fields':['department','course']})
    ]

class CourseAdmin(admin.ModelAdmin):
    list_display = ['c_id','c_name']
    list_display_links = ['c_id','c_name']
    list_per_page = 5

admin.site.register(Department,DepartmentAdimin)
admin.site.register(Student,StudentAdimin)
admin.site.register(Course,CourseAdmin)
```

### `auth`系统

#### `User`用户

##### 创建用户：

```
from django.contrib.auth.models import User
User.objects.create_user(username=username,password=password,email=email)
```

##### 验证用户：

```
from django.contrib.auth import authenticate
user = authenticate(username=username,password=password)
if user is not None:
    # 这个用户存在数据库中
else:
    # 这个用户没有存在这个数据库中
```

##### 登录

```
from django.contrib.auth import authenticate, login

def my_view(request):
    username = request.POST['username']
    password = request.POST['password']
    user = authenticate(username=username, password=password)
    if user is not None:
        if user.is_active:
            login(request, user)
            # 登录成功
        else:
            # 用户没有被激活，不能登录
    else:
        # 用户名或者密码错误
```

##### 注销

```
from django.contrib.auth import logout
  
  def logout_view(request):
      logout(request)
      # 注销这个用户。他的session信息将被清除掉。
```

`login_required`装饰器

```
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...
```

如果没有登录成功，会跳转到`settings.LOGIN_URL`指定的URL中。否则，直接执行函数中的内容。

##### `User`模型常用属性和方法

- `username`：用户名。
- `email`：邮箱。
- `groups`：多对多的组。
- `user_permissions`：多对多的用户权限。
- `is_staff`： 是否是`admin`的管理员。
- `is_active`： 是否激活，判断该用户是否可用。
- `is_superuser`: 是否是超级用户。
- `last_login`： 上次登录时间。
- `date_joined`： 注册时间。
- `is_authenticated`： 是否验证通过了。
- `is_anonymous`：是否是匿名用户。
- `set_password(raw_password)`： 设置密码，传原生密码进去。
- `check_password(raw_password)`： 检查密码。
- `has_perm(perm)`： 判断用户是否有某个权限。
- `has_perms(perm_list)`： 判断用户是否有权限列表中的某个列表

#### `Permission`权限模型

##### 在模型中添加权限

```
from django.db import models

class BlogModel(models.Model):
    id = models.AutoField(primary_key=True)
    title = models.CharField(max_length=100,blank=True)
    content = models.TextField()

    class Meta:
        permissions = (
            ('watch_article', u'查看文章的权限'),
            ('update_article', u'修改文章的权限'),
            ('delete_article', u'删除文章的权限'),
            ('add_article', u'发布文章的权限'),
        )
```

##### 在代码中添加权限

```
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType
def test(request):
    content_type = ContentType.objects.get_for_model(BlogModel)
    permission = Permission.objects.create(
        codename='can_publish',
        name='Can Publish BlogMoModel',
        content_type=content_type,
    )
    permission.save()
    return HttpResponse('success')
```

##### 用户权限操作

```
myuser.user_permissions.set([permission_list])
myuser.user_permissions.add(permission, permission, ...)
myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()
myuser.has_perm('foo.add_bar')
```

访问权限的方式：`appname`+`.`+`权限名称`。

#### `Group`模型

- 所属包`django.contrib.auth.models.Group`
- 创建`Group`：必须传一个`name`参数进去。
- `Group`操作：

```
group.permissions.set([permission_list])
group.permissions.add(permission, permission, ...)
group.permissions.remove(permission, permission, ...)
group.permissions.clear()
```


##代码

```python
#----ts22\views.py-----

from django.shortcuts import render,redirect,reverse
from django.http import HttpResponse
from .forms import AddForm,RegisterForm,LoginForm
import json
from .models import (UserModel)
# Create your views here.
from django.contrib.auth.models import User, Permission, Group
from django.contrib.auth import authenticate,login,logout

def add(request):
    """01利用form创建前端代码和校验"""
    if request.method == 'GET':
        form = AddForm()
        return render(request,'add_form.html',{"form":form})
    elif request.method =='POST':
        form = AddForm(request.POST)    #拿到form提交的数据
        if form.is_valid():
            print(form.cleaned_data)    #{'second': 3, 'first': 2}
            #字典获取值的方法，两种[]和get
            first = form.cleaned_data['first']
            second = form.cleaned_data.get('second')

            return HttpResponse(str(int(first)+int(second)))


def register(request):
    """03注册"""
    if request.method == 'GET':
        form = RegisterForm()
        return render(request,'register.html',{'form':form})
    elif request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            password_repeat = form.cleaned_data.get('password_repeat')
            email = form.cleaned_data.get('email')
            if password == password_repeat:
                # user = UserModel()
                # user.username = username
                # user.password = password
                # user.email = email
                # user.save()
                #不用自己创建的UserModel了，直接使用auth里面的User模型
                User.objects.create_user(username=username,password=password,email=email)
                return HttpResponse("注册成功")
            # data = {
            #     'username':username,
            #     'password':password,
            #     'password_repeat':password_repeat,
            #     'email':email
            # }
            # return HttpResponse(json.dumps(data))
            else:
                return HttpResponse('注册失败')


def login_aps(request):
    """04登录"""
    if request.method == 'GET':
        return render(request,'login22.html')
    elif request.method == 'POST':
        form = LoginForm(request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            # user = UserModel.objects.filter(username=username,password=password)

            #使用auth自带的authenticate自带的方法验证
            user = authenticate(username=username,password=password)
            if user:
                # request.session['username']=username
                #使用auth自带的login登录，
                # 会在客户端生成session_id；服务端的django_session表中生成session的内容
                login(request,user)
                nexturl = request.GET.get('next')
                print(nexturl)
                #/blog/
                if nexturl:
                    # return redirect(reverse('ts22_home'))
                    return redirect(nexturl)
                else:
                    return redirect(reverse('ts22_home'))
            else:
                return redirect(reverse('ts22_register'))
        else:
            return render(request,'login22.html',{'error':form.errors})


def home(request):
    """05home页面"""
    # a = int('xxxxx')  
    #故意写错，来验证我们自定义的异常中间件
    # print(request.myuser)
    username = request.myuser
    #把request.user换成request.myuser来验证我们自定义的上下文
    # username = request.session.get('username','未登录')
    return render(request,'home22.html',{'username':username})


def logout_aps(request):
    """06 退出"""
    # request.session.flush()   #删除cookies的id,并且删除服务端session表的记录
    logout(request) #使用django auth系统自带的login,删除session_id并且session表记录
    return redirect(reverse('ts22_home'))

#-----------------------------------------------------
#现在用系统自带的User


def test(request):
    """07 多个交互的操作"""
    # User.objects.create_user(username='xiaoming',password='qwe123',email='43992163.com')
    #为普通用户kakaxi添加可以add Blogmodel的权限
    # kakaxi = User.objects.filter(username='kakaxi').first()
    # print(kakaxi)
    # permission = Permission.objects.filter(codename='add_blog').first()
    # print(permission)
    #blog | blog | Can add blog
    # kakaxi.user_permissions.add(permission)
    # 组操作:
    # g1 = Group.objects.create(name='user_add_blog')
    # g1.save()
    #给组添加权限
    # g1 = Group.objects.filter(name='user_add_blog').first()
    # permission = Permission.objects.filter(codename='add_blog').first()
    # g1.permissions.add(permission)
    # g1.save()
    #把用户mingren1添加到group组里面,那么组员就具有了组的权限
    mingren1 = User.objects.filter(username='mingren1').first()
    g1 = Group.objects.filter(name='user_add_blog').first()
    g1.user_set.add(mingren1)
    return HttpResponse('添加数据成功')
```



#### ts22\models.py

```python
from django.db import models

# Create your models here.


class UserModel(models.Model):
    username = models.CharField(max_length=30,unique=True)
    password = models.CharField(max_length=30)
    email = models.EmailField()
    def __str__(self):
        return self.username
```





#### ts22\forms.py

```python
from django import forms


class AddForm(forms.Form):
    first = forms.IntegerField()
    second = forms.IntegerField()


class RegisterForm(forms.Form):
    username = forms.CharField(max_length=8,min_length=6)
    password = forms.CharField(max_length=8,min_length=6,
                               widget=forms.PasswordInput(
                                   attrs={"placeholder":"请输入密码"}),
                               error_messages={'min_length':'密码长度不能小于6',
                                               'max_length':'密码长度不能超过8'})
    password_repeat = forms.CharField(max_length=8,
                                      widget=forms.PasswordInput(
                                          attrs={'placeholder':'请在输入密码'}),
                                      error_messages={'max_length':'密码长度小于6'})
    email = forms.EmailField()


class LoginForm(forms.Form):
    username = forms.CharField(label='用户名',max_length=8,
                               error_messages={'max_length':'不能超过8个字符'})
    password = forms.CharField(label='密码',max_length=8,min_length=6,
                               widget=forms.PasswordInput(
                                   attrs={'placeholder':'请输入密码'}
                               ))
```



####自定义上下文

```python
#---mycontextprocess.py(与settings.py同级)------------------

from ts22.models import UserModel


def myuser(request):
    username = request.session.get('username','未登录')
    # user = UserModel.objects.filter(username=username).first()
    user = request.user
    if user:
        return {'myuser':user}
    else:
        return {}
```





####自定义中间件

```python
#---mymiddleware.py--(与settings.py同级)----

from django.http import HttpResponse
from django.utils.deprecation import MiddlewareMixin

#第一种方式



class MyException(MiddlewareMixin):
    def process_exception(self,request,exception):
        return HttpResponse(exception)


from ts22.models import UserModel
#第二种方式
class UserMiddleware(MiddlewareMixin):
    def __init__(self,get_response):
        self.get_response = get_response

    def __call__(self, request):
        #request到达view之前执行的代码
        username = request.session.get('username','未登录')
        # user = UserModel.objects.filter(username=username).first()
        user = request.user
        # print('--->',user)
        if user:
            if not hasattr(request,'myuser'):
                setattr(request,'myuser',user)  #request.myuser自定义一个属性
        else:
            setattr(request,'myuser','游客')


        response = self.get_response(request)  #这一行代码实际是分割的作用
        # response到达用户浏览器之前执行的代码
        return response
```



```python
#-----settings.py-------

#_*_ coding:utf-8 _*_
"""
Django settings for hello_django project.

Generated by 'django-admin startproject' using Django 1.11.7.

For more information on this file, see
https://docs.djangoproject.com/en/1.11/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/1.11/ref/settings/
"""

import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/1.11/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'r80)#ne^8&6vtjdz#jxark!h&w!!y6#j+rqf7*we-usl)ipxi0'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ['*']


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'book',
    'music',
    'move',
    'basketball',
    'blog',
    'ts11',
    'ts22',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # 'hello_django.mymiddleware.MyException',    #自定义异常中间件
    'hello_django.mymiddleware.UserMiddleware', #自定义User中间件
]

ROOT_URLCONF = 'hello_django.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'hello_django.mycontextprocessor.myuser'#自定义上下文
                # (其实就是给template一个变量)
            ],
        },
    },
]

WSGI_APPLICATION = 'hello_django.wsgi.application'


# Database
# https://docs.djangoproject.com/en/1.11/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',#数据库引擎
        'NAME': 'my_django', #数据库名称
        'USER': 'mhh123', # 链接数据库的用户名
        'PASSWORD': 'mhh123', # 链接数据库的密码
        'HOST': '39.105.77.167', # mysql服务器的域名和ip地址
        'PORT': '3306', # mysql的一个端口号,默认是3306
        'OPTIONS':{
            # 'init_command':"SET sql_mode='STRICT_TRANS_TABLES'",
        'sql_mode': 'traditional', 
        }
    }
}


# Password validation
# https://docs.djangoproject.com/en/1.11/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/1.11/topics/i18n/

LANGUAGE_CODE = 'zh-Hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.11/howto/static-files/

STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR,'static')
]
#LOGIN_URL='ts22/login'会报错
#http://mhh4399.com:8000/blog/ts22/login?next=/blog/
#解决办法LOGIN_URL='/ts22/login'
LOGIN_URL='/ts22/login'

```





#### blog\views.py

```
from django.shortcuts import render,redirect,reverse
from django.http import HttpResponse
from .models import Blog
from django.contrib.auth.decorators import login_required,permission_required
# Create your views here.

#这个是用来验证没有登录，跳转到settings.py中设置的默认LOGIN_URL="这里设置的是app-blog的index页面"
@login_required
def index(request):
    """01index页面"""
    return render(request,"demo_index.html")

#这个是用来验证给视图加权限，如果该已登录的用户有权限，
#就继续执行函数，没有就默认给跳转到了用户登录页面
#可以观察一下，下面的表，因为django创建项目的时候会有哪些表，是什么关系
@permission_required('blog.add_blog')  #blog是模型名字
def blog_add(request):
    """02添加文章"""
    if request.method == "POST":
        title = request.POST.get("title",None)
        content = request.POST.get("content",None)
        blog = Blog()
        #blog = Blog(title=title,content=content)
        #可以直接实例化并赋值,一步到位
        blog.title=title
        blog.content=content
        blog.save()
        return HttpResponse("文章添加成功")

    return render(request,"demo_add.html")

```



#### django创建项目生成的表



**仔细观察django为我们创建了那些表**



![](http://p7bj6aatj.bkt.clouddn.com/auth_%E7%B3%BB%E7%BB%9F%E8%87%AA%E5%B8%A6%E7%9A%84%E8%A1%A8.png)