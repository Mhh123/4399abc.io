## Flask-Session&Flask-SQLAlchemy

#### flask-session		(给flask app 添加服务端会话功能)

是flask框架的session组件，由于原来flask内置session使用签名cookie保存，该组件则将支持session保存到多个地方。

如:

- redis：保存数据的一种工具，五大类型。非关系型数据库
- memcached
- filesystem
- mongodb
- sqlalchmey：那数据存到数据库表里面



~~~
class Session(object):
    """This class is used to add Server-side Session to one or more Flask
    applications.

    There are two usage modes.  One is initialize the instance with a very
    specific Flask application::

        app = Flask(__name__)
        Session(app)

    The second possibility is to create the object once and configure the
    application later::

        sess = Session()

        def create_app():
            app = Flask(__name__)
            sess.init_app(app)
            return app

    By default Flask-Session will use :class:`NullSessionInterface`, you
    really should configurate your app to use a different SessionInterface.
~~~







#### flask-sqlalchemy		(给flask applications 集成sqlalchemy的功能)

此类用于控制SQLAlchemy与一个或多个Flask应用程序的集成。根据您初始化对象的方式，它可以立即使用，或者根据需要附加到Flask应用程序

API:

http://www.pythondoc.com/flask-sqlalchemy/api.html#flask.ext.sqlalchemy.SQLAlchemy





配置

~~~
class flask.ext.sqlalchemy.SQLAlchemy（app = None，use_native_unicode = True，session_options = None，metadata = None，query_class = <class'stars_sqlalchemy.BaseQuery'>，model_class = <class'stars_sqlalchemy.Model'> ）
~~~

有两种使用模式非常相似。一个是将实例绑定到一个非常特定的Flask应用程序：

~~~
app  =  Flask （__name__ ）
db  =  SQLAlchemy （app ）
~~~

第二种可能性是创建一次对象并稍后配置应用程序以支持它：

~~~
db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    return app
~~~

两者之间的区别是，在第一种情况的方法，如 [`create_all()`](http://www.pythondoc.com/flask-sqlalchemy/api.html#flask.ext.sqlalchemy.SQLAlchemy.create_all)与[`drop_all()`](http://www.pythondoc.com/flask-sqlalchemy/api.html#flask.ext.sqlalchemy.SQLAlchemy.drop_all)将所有的工作时间，但在第二种情况下`flask.Flask.app_context()`必须存在。

~~~python
class SQLAlchemy(object):
    """This class is used to control the SQLAlchemy integration to one
    or more Flask applications.  Depending on how you initialize the
    object it is usable right away or will attach as needed to a
    Flask application.

    There are two usage modes which work very similarly.  One is binding
    the instance to a very specific Flask application::

        app = Flask(__name__)
        db = SQLAlchemy(app)

    The second possibility is to create the object once and configure the
    application later to support it::

        db = SQLAlchemy()

        def create_app():
            app = Flask(__name__)
            db.init_app(app)
            return app
~~~







安装

`pip install flask-session`



`pip install flask-sqlalchemy`



#### **---settings.py---**

~~~python
class Config(object):
    SECRET_KEY = 'super secret key' #這個是配置一個密鑰（使用session就要配置密鑰）
    SESSION_TYPE = 'redis'  #這個是配置session存儲在哪
    PERMANENT_SESSION_LIFETIME = 60 #這個是配置session的存活時間
    DATABASE_URI = {
        'dialect': 'mysql',     #数据库
        'driver': 'pymysql',    #驱动
        'username': 'mhh123',
        'password': 'mhh123',
        'host': '127.0.0.1',
        'port': 3306,
        'databases': 'my_flask' #数据库名
    }
    # 'SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://mhh123:mhh123@127.0.0.1:3306/my_flask'
    SQLALCHEMY_DATABASE_URI = '{}+{}://{}:{}@{}:{}/{}'.format(
        DATABASE_URI['dialect'], DATABASE_URI['driver'],
        DATABASE_URI['username'], DATABASE_URI['password'],
        DATABASE_URI['host'], DATABASE_URI['port'],
        DATABASE_URI['databases']
    )
    DEBUG = False
    TESTING = False


class Develop(Config):
    DEBUG = True


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





#### **---myapp\ext.py---**

~~~python
from flask_session import Session
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def init_ext(app):
    Session(app=app)
    db.init_app(app)

~~~



#### **---views\__init__.py---**

~~~python
from my_app.ext import init_ext		#加session和sqlalchemy
from my_app.settings import env		#配置
from my_app.views import blue		#蓝图名
from my_app.views.Sessiondemo import session_blue


from flask import Flask

from my_app.views.mode_view import model_blue
from my_app.views.templateviews import template_blue




def create_app(param):
    app = Flask(__name__)
    #app必须要注册一个蓝图才能使用
    app.register_blueprint(blueprint=blue)	#注册蓝图
    app.register_blueprint(blueprint=session_blue)
    app.register_blueprint(blueprint=template_blue)
    app.register_blueprint(blueprint=model_blue)

    app.config.from_object(env.get(param))
    init_ext(app)
    return app			#返回一个加了很多功能的app
~~~





#### 例子1:

**---views\Sessiondemo.py---**

~~~python
from flask import Blueprint, session, render_template, redirect, url_for, request

session_blue = Blueprint('session_blue', __name__)

@session_blue.route('/writesession/')
def write_session():



    session['age'] = 18 #注意session是一个全局的变量，是我们from flask import session导入的
    # 没有flask_session装饰之前，这里的session是写在服务器缓存里面的
    return '写入成功'


@session_blue.route('/login/')
def login():
    return render_template('Login.html')


@session_blue.route('/loginsuccess/', methods=['GET', 'POST'])
def login_success():
    username = request.form.get('username')
    session['username'] = username
    return redirect(url_for('session_blue.welcome'))


@session_blue.route('/welcome/')
def welcome():
    print(session)  # < RedisSession{'_permanent': True} >
    print(type(session))  # < class 'werkzeug.local.LocalProxy'>
    return render_template('welcome.html', username=session['username'])

~~~





**---templates\Login.html---**

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
<form action="/loginsuccess/" method="post">
    <input type="text" name="username" placeholder="请输入用户名">
    <input type="submit" name="submit" value="提交">
</form>
</body>
</html>
~~~



**---templates\welcome.html---**

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>好久不见欢迎回来{{ username }}</h1>
</body>
</html>
~~~



运行

`python manager.py runserver -d -r -h 0.0.0.0 -p 8000`

访问  http://mhh4399.com:8000/writesession/

结果  写入成功

查看 redis已经有了数据，虽然get一大堆，但是还是能找到age

~~~
127.0.0.1:6379> keys *
1) "session:011aa733-693a-4e8e-aac2-8a3e197048de"
2) "_kombu.binding.celery"
127.0.0.1:6379> get "session:011aa733-693a-4e8e-aac2-8a3e197048de"
"\x80\x03}q\x00(X\x03\x00\x00\x00ageq\x01K\x12X\n\x00\x00\x00_permanentq\x02\x88u."

~~~



访问  http://mhh4399.com:8000/login/

输入 mingren

之后被重定向到  http://mhh4399.com:8000/welcome/

页面显示 

好久不见mingren

查看 redis已经有了数据，虽然get一大堆，但是还是能找到username

~~~
127.0.0.1:6379> keys *
1) "session:f4361935-4031-47d4-95d5-2551dc495092"
2) "_kombu.binding.celery"
127.0.0.1:6379> get "session:f4361935-4031-47d4-95d5-2551dc495092"
"\x80\x03}q\x00(X\b\x00\x00\x00usernameq\x01X\a\x00\x00\x00mingrenq\x02X\n\x00\x00\x00_permanentq\x03\x88u."
~~~





#### 例子2:

---

**---views\models.py---**

~~~python
from my_app.ext import db



class Student(db.Model):
    id = db.Column(db.Integer,primary_key=True,autoincrement=True)
    s_name = db.Column(db.String(32),nullable=False,unique=True)
    s_age = db.Column(db.Integer,nullable=False,default=18)
    s_score = db.Column(db.Float,nullable=False,default=0)

~~~



**---views\mode_view.py---**

~~~python
from flask import Blueprint, render_template
import random

from my_app.ext import db
from my_app.views.models import Student

model_blue = Blueprint('model_blue',__name__)

@model_blue.route('/createmodel/')
def create_tale():
    db.create_all()
    return '创建表格成功'

@model_blue.route('/add_student/')
def add_student():
    stu = Student()
    stu.s_name = '小明'
    stu.s_age = 16
    stu.s_score = 80
    db.session.add(stu) #添加单个学生
    db.session.commit()
    return '添加成功'

@model_blue.route('/drop_table/')
def drop_table():
    db.drop_all()
    return '删除表格成功'

@model_blue.route('/add_students/')
def add_students():
    students = []
    for i in range(10):
        stu = Student()
        num = random.randrange(100)
        stu.s_name = 'xiaojun%d'% num   #这里拼接一个随机num是因为s_name我们设置了unique=True
        stu.s_age = num
        stu.s_score = num +60

        students.append(stu)
        #下面的两行代码是单个stu添加的
        # db.session.add(stu)
        # db.session.commit()
    db.session.add_all(students)
    db.session.commit()
    return '批量添加学生成功'


@model_blue.route('/get_all_student/')
def get_all_students():
    students = Student.query.all()  #查询所有
    return render_template('student_list.html',students=students)
~~~





访问  http://mhh4399.com:8000/createmodel/

页面  

​	创建表格成功

查看数据库

~~~mysql
mysql> show tables;
Empty set (0.00 sec)

mysql> show tables;
+--------------------+
| Tables_in_my_flask |
+--------------------+
| student            |
+--------------------+
1 row in set (0.00 sec)

mysql> desc student;
+---------+-------------+------+-----+---------+----------------+
| Field   | Type        | Null | Key | Default | Extra          |
+---------+-------------+------+-----+---------+----------------+
| id      | int(11)     | NO   | PRI | NULL    | auto_increment |
| s_name  | varchar(32) | NO   | UNI | NULL    |                |
| s_age   | int(11)     | NO   |     | NULL    |                |
| s_score | float       | NO   |     | NULL    |                |
+---------+-------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

~~~





访问  http://mhh4399.com:8000/add_student/

页面

​	添加成功

查看数据库的student表

~~~mysql
mysql> select * from student;
+----+--------+-------+---------+
| id | s_name | s_age | s_score |
+----+--------+-------+---------+
|  1 | 小明   |    16 |      80 |
+----+--------+-------+---------+
1 row in set (0.00 sec)

~~~







访问  http://mhh4399.com:8000/add_students/

页面

​	批量添加学生成功

查看数据库student表

~~~mysql
mysql> select * from student;
+----+-----------+-------+---------+
| id | s_name    | s_age | s_score |
+----+-----------+-------+---------+
|  1 | 小明      |    16 |      80 |
|  2 | xiaojun24 |    24 |      84 |
|  3 | xiaojun39 |    39 |      99 |
|  4 | xiaojun5  |     5 |      65 |
|  5 | xiaojun53 |    53 |     113 |
|  6 | xiaojun66 |    66 |     126 |
|  7 | xiaojun74 |    74 |     134 |
|  8 | xiaojun48 |    48 |     108 |
|  9 | xiaojun82 |    82 |     142 |
| 10 | xiaojun45 |    45 |     105 |
| 11 | xiaojun57 |    57 |     117 |
+----+-----------+-------+---------+
11 rows in set (0.00 sec)

~~~







访问   http://mhh4399.com:8000/get_all_student/

页面

~~~
all_studnets
姓名:小明 年纪:16成绩:80.0
姓名:xiaojun24 年纪:24成绩:84.0
姓名:xiaojun39 年纪:39成绩:99.0
姓名:xiaojun5 年纪:5成绩:65.0
姓名:xiaojun53 年纪:53成绩:113.0
姓名:xiaojun66 年纪:66成绩:126.0
姓名:xiaojun74 年纪:74成绩:134.0
姓名:xiaojun48 年纪:48成绩:108.0
姓名:xiaojun82 年纪:82成绩:142.0
姓名:xiaojun45 年纪:45成绩:105.0
姓名:xiaojun57 年纪:57成绩:117.0
~~~



访问  http://mhh4399.com:8000/drop_table/

页面

​	删除表格成功

查看数据库

~~~mysql
mysql> show tables;
Empty set (0.00 sec)
~~~

