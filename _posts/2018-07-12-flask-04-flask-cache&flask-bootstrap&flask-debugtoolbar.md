## flask-bootstrap



#### 安装

`pip install flask-bootstrap`

#### 配置

###### ---ext.py---

~~~python
from flask_bootstrap import Bootstrap

def init_ext(app):
	Bootstrap(app)
~~~





## flask-debugtooltar



#### 安装

`pip install flask-debugtoolbar`



#### 配置



###### --ext.py---

~~~python
from flask_debugtoolbar import DebugToolbarExtension

def init_ext(app):
	toolbar = DebugToolbarExtension(app)
~~~

###### ---setting---

~~~python
DEBUG = True
    #使用flask-debugtoolbar需要打开debug开关
~~~





## flask-cache



#### 安装

`pip install flask-cahce`



#### 配置

###### ---ext.py---

~~~python
from flask_cache import Cache

cache = Cache(config={'CACHE_TYPE':'redis'})

def init_ext(app):
	cache.init_app(app=app)
~~~









## 项目结构

~~~
flaskday04
├── app
│   ├── ext.py
│   ├── __init__.py
│   ├── models.py
│   ├── __pycache__
│   │   ├── ext.cpython-35.pyc
│   │   ├── __init__.cpython-35.pyc
│   │   ├── models.cpython-35.pyc
│   │   └── settings.cpython-35.pyc
│   ├── settings.py
│   ├── static
│   ├── templates
│   │   ├── hello.html
│   │   ├── index.html
│   │   └── show_students.html
│   └── views
│       ├── __init__.py
│       ├── mytest.py
│       └── __pycache__
│           ├── __init__.cpython-35.pyc
│           └── mytest.cpython-35.pyc
├── manager.py
└── migrations
    ├── alembic.ini
    ├── env.py
    ├── __pycache__
    │   └── env.cpython-35.pyc
    ├── README
    ├── script.py.mako
    └── versions
        ├── f2ad6df382ff_.py
        └── __pycache__
            └── f2ad6df382ff_.cpython-35.pyc

~~~









---

###### ---manager.py---

~~~python
from flask_migrate import MigrateCommand
from flask_script import Manager

from app import create_app

app = create_app('develop')
# 创建一个Manager对象
manager = Manager(app)
manager.add_command('db', MigrateCommand)   # 添加Migrate的所有子命令到db下

if __name__ == '__main__':
    manager.run()

~~~



###### ---app\__init__.py---

~~~python
from app.ext import init_ext
from app.settings import env

from .views.mytest import test_blue

from flask import Flask


def create_app(param):
    app = Flask(__name__)
    app.config.from_object(env.get(param))  # 建议把配置项放在前面；避免有时候放后面，配置项不起效
    # app必须要注册一个蓝图才能使用
    app.register_blueprint(blueprint=test_blue)


    init_ext(app)
    return app

~~~



###### ---app\ext.py---

~~~python
from flask_bootstrap import Bootstrap
from flask_cache import Cache
from flask_debugtoolbar import DebugToolbarExtension
from flask_migrate import Migrate
from flask_session import Session
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
cache = Cache(config={'CACHE_TYPE':'redis'})

def init_ext(app):
    Session(app=app)
    db.init_app(app)
    migrate = Migrate(app=app, db=db) ## 用来绑定app和db到flask_migrate的
    Bootstrap(app)
    toolbar = DebugToolbarExtension(app)
    cache.init_app(app=app)

~~~

###### ---app\settings.py---

~~~python
class Config(object):
    SECRET_KEY = 'super secret key'  # 這個是配置一個密鑰（使用session就要配置密鑰）
    SESSION_TYPE = 'redis'  # 這個是配置session存儲在哪
    PERMANENT_SESSION_LIFETIME = 60  # 這個是配置session的存活時間
    DATABASE_URI = {
        'dialect': 'mysql',  # 数据库
        'driver': 'pymysql',  # 驱动
        'username': 'mhh123',
        'password': 'mhh123',
        'host': '127.0.0.1',
        'port': 3306,
        'databases': 'my_flask04'  # 数据库名
    }
    # 'SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://mhh123:mhh123@127.0.0.1:3306/my_flask'
    SQLALCHEMY_DATABASE_URI = '{}+{}://{}:{}@{}:{}/{}'.format(
        DATABASE_URI['dialect'], DATABASE_URI['driver'],
        DATABASE_URI['username'], DATABASE_URI['password'],
        DATABASE_URI['host'], DATABASE_URI['port'],
        DATABASE_URI['databases']
    )
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    DEBUG = False
    TESTING = False


class Develop(Config):
    DEBUG = False
    #使用flask-debugtoolbar需要打开debug开关


class Test(Config):
    TESTING = True


class Product(Config):
    DEBUG = False
    TESTING = False


env = {
    'develop': Develop,
    'test': Test,
    'product': Product
}

~~~



#### 例子1



###### ---views\mytest.py---

~~~python
import random

from flask import Flask, Blueprint, config, current_app, g, render_template, request, abort, session

from app.ext import cache

from app.models import Student

from app.ext import db

test_blue = Blueprint('test_blue', __name__)


@test_blue.route('/')
def hello_world():
    """01 config和g的演示"""
    # 在代码理获取配置项，需要使用current_app
    # 然后.config
    # print(current_app.config)
    # <Config {'SQLALCHEMY_ECHO': False,
    # 省略...
    #  'SESSION_TYPE': 'redis', 'JSONIFY_PRETTYPRINT_REGULAR': False}>


    # g
    # 一个独立的全局变量(注意'独立'代表的是每一个request)
    return render_template('hello.html')


@test_blue.route('/add_students/')
def add_students():
    """  02造数据  """
    students = []
    for i in range(10):
        stu = Student()
        num = random.randrange(50)
        stu.s_name = '小芳%d' % num
        stu.s_age = 17
        stu.s_score = 50 + num
        students.append(stu)

    db.session.add_all(students)
    db.session.commit()
    return '批量添加学生成功'


def make_cache_key():
    """  03  返回一个唯一的远程地址 """
    page_num = request.args['page_num']
    return request.remote_addr + 'page_num = %s' % page_num

#key_prefix  注意这个的作用是为了避免所有的请求都返回原来的缓存
#我们结合make_cache_key是为了获取remote_addr，实现每一个浏览器有一个唯一
#同时也解决而原始缓存不更新的问题

@test_blue.route('/paginationstudent/')
@cache.cached(timeout=10, key_prefix=make_cache_key)  # 注意cache装饰器要写在route装饰器的下面
#我们加cache装饰器是为了，避免每次访问每次重新查询，减轻数据库的压力
def paginate_student():
    """  04 paginate来实现分页"""
    print('请求数据')
    page_num = int(request.args['page_num'])
    per_page = 5


    #利用paginate来实现分页
    pagination = Student.query.paginate(page=page_num, per_page=per_page)
    print('--->', pagination)
    return render_template('index.html', pagination=pagination)


@test_blue.route('/get_all_students/')
def get_all_students():
    """05  获取到所有的学生"""
    # 拿数据
    students = cache.get('students')
    # students = session.get('students')
    if not students:
        # 如果没有就去数据库查询
        print('设置redis')
        students = Student.query.all()
        cache.set('students', students, timeout=30)

        # session['student']=students
    """
        我们来讨论一下，session 和 cache 
        #区别:
        #session 1个key,不可以设置单个过期时间;
        #cache两个key,可以设置单个过期时间
        
        如果使用session['student']=studnets，在redis里面有一个key
        1) "session:e2f56fc5-786f-4549-a4e5-f587ff7f8641"
        如果使用cache.set('students',students,timeout=30)
        在redis里面有两个key
        1) "session:f4f4ab29-4446-4109-bbff-61749d52012f"
        2) "flask_cache_students"
        
        127.0.0.1:6379> get "session:f4f4ab29-4446-4109-bbff-61749d52012f"
        "\x80\x03}q\x00X\n\x00\x00\x00_permanentq\x01\x88s."
        127.0.0.1:6379> get "flask_cache_students"
        "!\x80\x03]q\x00(capp.models\nStudent\nq\x01)\x81q\x02}
        q\x03(X\x12\x00\x00\x00_sa_instance_stateq\x04csqlalchemy.orm
        ......
        8a\xb33rA\x01\x00\x00h$G@J\x80\x00\x00\x00\x00\x00ube."
    """

    return render_template('show_students.html', students=students)

@test_blue.before_request       #中间件
def handler_before():
    """06  中间件，限制访问频率"""
    count = cache.get('count') or 1
    if count > 10:
        abort(404)
        return '5秒钟之内只能访问10次'
    count += 1
    cache.set('count', count, timeout=5)


~~~





###### ---app\templates\index.html---

~~~html
{% extends '/bootstrap/base.html' %}

{% block content %}
  <nav aria-label="Page navigation">

    <ul>
      {% for student in pagination.items %}
        <li>id:{{ student.id }} 姓名:{{ student.s_name }} 年龄:{{ student.s_age }} 成绩:{{ student.s_score }}</li>
      {% endfor %}

    </ul>

    {#  下面显示的页码 #}
    <ul class="pagination">
      {#  如果有上一页  #}
      {% if pagination.has_prev %}

        <li>
          <a href="{{ url_for('test_blue.paginate_student') }}?page_num={{ pagination.prev_num }}" aria-label="Previous">
            <span aria-hidden="true">&laquo;</span>
          </a>
        </li>
      {% else %}
        <li>
          <a href="#" aria-label="Previous">
            <span aria-hidden="true">&laquo;</span>
          </a>
        </li>
      {% endif %}



      {#      不推荐下面的写法#}

      {#      {% for page in range(pagination.pages) %}#}
      {#        <li><a href="#">{{ page  + 1 }}</a></li>#}
      {#      {% endfor %}#}

      {% for page in pagination.iter_pages() %}
        {% if page %}

          {# 如果现在选择的不是当前页 #}
          {% if page != pagination.page %}
            <li><a href=" {{ url_for('test_blue.paginate_student') }}?page_num={{ page }} ">{{ page }}</a></li>
          {% else %}
            <li class='active'><a href="#">{{ page }}</a></li>
          {% endif %}


        {% else %}
          <span class=ellipsis>…</span>
        {% endif %}
      {% endfor %}


      {% if pagination.has_next %}
        <li>
          <a href="{{ url_for('test_blue.paginate_student') }}?page_num={{ pagination.next_num }}" aria-label="Next">
            <span aria-hidden="true">&raquo;</span>
          </a>
        </li>
        {#      {% else %}#}
        {#        <li>#}
        {#          <a href="#" aria-label="Next">#}
        {#            <span aria-hidden="true">&raquo;</span>#}
        {#          </a>#}
        {#        </li>#}
      {% endif %}


    </ul>
  </nav>


{% endblock %}
~~~

