## Flask-Migrate

**Flask-Migrate**是一个扩展，使用Alembic处理Flask应用程序的SQLAlchemy数据库迁移。数据库操作通过Flask命令行界面或Flask-Script扩展提供。



问题思考：

​	我们已经有了db.create_all()和db.drop_all()，我们为什么要用flask-migrate呢？

## 安装

​	`pip install flask-migrate`



#### 项目结构

~~~
(fkpy3) kakaxi@ubuntu64:~/flaskcode$ tree -a
.
├── flaskday03
│   ├── app
│   │   ├── ext.py
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── __pycache__
│   │   │   ├── ext.cpython-35.pyc
│   │   │   ├── __init__.cpython-35.pyc
│   │   │   ├── models.cpython-35.pyc
│   │   │   └── settings.cpython-35.pyc
│   │   ├── settings.py
│   │   ├── static
│   │   ├── templates
│   │   │   └── showstudent.html
│   │   └── views
│   │       ├── fav_view.py
│   │       ├── __init__.py
│   │       ├── person_view.py
│   │       ├── __pycache__
│   │       │   ├── fav_view.cpython-35.pyc
│   │       │   ├── __init__.cpython-35.pyc
│   │       │   ├── person_view.cpython-35.pyc
│   │       │   └── studentview.cpython-35.pyc
│   │       └── studentview.py
│   ├── manager.py
│   └── migrations
│       ├── alembic.ini
│       ├── env.py
│       ├── __pycache__
│       │   └── env.cpython-35.pyc
│       ├── README
│       ├── script.py.mako
│       └── versions
│           ├── 9b34317723a1_.py
│           ├── cb90fa29d773_.py
│           ├── fb0acf0ea22b_.py
│           └── __pycache__
│               ├── 9b34317723a1_.cpython-35.pyc
│               ├── cb90fa29d773_.cpython-35.pyc
│               └── fb0acf0ea22b_.cpython-35.pyc

~~~



## 配置

- Migrate(app=app, db=db) # 用来绑定app和db到flask_migrate的
- manager.add_command('db', MigrateCommand)   # 添加Migrate的所有子命令到db下

#### ---ext.py---

~~~python

from flask_migrate import Migrate
from flask_session import Session
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()


def init_ext(app):
    Session(app=app)
    db.init_app(app)
    Migrate(app=app, db=db) # 用来绑定app和db到flask_migrate的

~~~



#### ---manager.py---

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

#### ---settings.py---

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
        'databases': 'my_flask03'  # 数据库名
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

## 使用

- `python manager.py db init`  创建一个alembic仓库
- `python manager.py db migrate`  生成迁移文件
- `python manager.py db upgrade`  将指定版本的迁移文件映射到数据库中
- `python manager.py runserver -r -h 0 -p 8000`  启动服务

#### 

#### ---views\__init__.py---

~~~python
from app.ext import init_ext
from app.settings import env

from .views.studentview import stu_blue
from .views.person_view import person_blue
from .views.fav_view import fav_blue

from flask import Flask


def create_app(param):
    app = Flask(__name__)
    app.config.from_object(env.get(param))  # 建议把配置项放在前面；避免有时候放后面，配置项不起效
    # app必须要注册一个蓝图才能使用
    app.register_blueprint(blueprint=stu_blue)
    app.register_blueprint(blueprint=person_blue)
    app.register_blueprint(blueprint=fav_blue)

    init_ext(app)
    return app

~~~



#### 例子1：

#### ---studentview.py---

~~~python
import random

from flask import Blueprint, render_template, request

from app.ext import db
from app.models import Student

stu_blue = Blueprint('stu_blue', __name__)


@stu_blue.route('/createtable/')
def create_table():
    db.create_all()
    return '创建表格成功'


@stu_blue.route('/droptable/')
def drop_table():
    db.drop_all()
    return '删除成功'


@stu_blue.route('/addstudents/')
def add_students():
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



@stu_blue.route('/get_all_students/')
def get_all_students():
    students = Student.query.all()
    return render_template('showstudent.html', students=students)


@stu_blue.route('/get_student_by_id/<int:s_id>/')
def get_student_by_id(s_id):
    """根据id拿到指定的数据"""
    stu = Student.query.get(s_id)
    return '拿到的学生姓名是{}'.format(stu.s_name)


@stu_blue.route('/get_student_by_condition/<name>')
def get_student_by_condition(name):
    students = Student.query.filter(Student.s_name == name)
    # gt lt ge le
    # contains  like  startswith endswith
    # in_([]) or_(( , )) and_ not_

    # filter_by
    return render_template('showstudent.html', students=students)


@stu_blue.route('/get_student_by_page/')
def get_student_by_page():
    """排序  如果组合使用需要先order_by"""
    # students = Student.query.order_by(Student.s_score)
    # students = Student.query.order_by(-Student.s_score) #降序
    # students = Student.query.offset(5)  #偏移(跳过制定个数的记录)
    # students = Student.query.limit(2)  #限制(显示显示几条记录)
    # students = Student.query.offset(5).limit(2)  #偏移5限制2
    # students = Student.query.order_by(Student.s_score).offset(5).limit(4)  #排序偏移5限制4

    page_num = int(request.args['page_num'])
    per_page = int(request.args['per_page'])

    # students = Student.query.limit(page_num).offset((per_page-1)*per_page)


    """
    paginate 返回值类型是一个Pagination对象，这个对象保存分页的相关信息
    
    pagination对象的items属性保存了获取到的这一页的所有数据
    
    """

    pagination = Student.query.paginate(page=page_num, per_page=per_page)
    # print(type(pagination)) #<class 'flask_sqlalchemy.Pagination'>
    students = pagination.items

    # print(pagination.page)
    # print(pagination.per_page)
    # print(pagination.query)
    # print(pagination.total)
    # print(pagination.has_prev)
    # print(pagination.has_next)

    # 打印
    # 2
    # 5
    # SELECT student.id AS student_id,
    # student.s_name AS student_s_name,
    # student.s_age  AS student_s_age,
    # student.s_score AS student_s_score FROM student
    # 10
    # True
    # False

    return render_template('showstudent.html', students=students)

~~~



#### 例子2：

#### ---person_view.py---

~~~python
import random

from app.models import Person, Address
from flask import Blueprint

from app.ext import db

person_blue = Blueprint('person_blue', __name__)


@person_blue.route('/add_persons/')
def add_persons():
    addrs = Address.query.all()
    provinces = []
    for add in addrs:
        provinces.append(add.a_province)

    persons = []
    for i in range(4):
        person = Person()

        num = random.randrange(10, 20)
        person.p_name = '张三%d' % num
        person.p_province = provinces.pop()
        persons.append(person)

    db.session.add_all(persons)
    db.session.commit()
    return '批量添加数据成功'


@person_blue.route('/get_person_by_id/<int:p_id>/')
def get_person_by_id(p_id):
    person = Person.query.get(p_id)
    # print(person.p_name)
    # print(person.p_province)
    # print(person.addr.a_shortname)
    # 张三18
    # 大陆2
    # 陆2

    return '返回一个人'

~~~



#### 例子3：

#### ---fav_view.py---

~~~python
from flask import Blueprint, request

from app.models import Favorite

from app.ext import db
from sqlalchemy import and_

fav_blue = Blueprint('fav_blue', __name__)


@fav_blue.route('/add_to_fav/')
def add_to_fav():
    """01添加收藏"""
    u_id = request.args['u_id']
    m_id = request.args['m_id']

    ex_fav = Favorite.query.filter(and_(Favorite.u_id == u_id,
                                        Favorite.m_id == m_id))
    """ex_fav 是一个BaseQuery对象
        有一个count()方法可以拿到总数
    """
    # print('--->',ex_fav.count())    #1
    if ex_fav.count() > 0:
        return '你已经收藏过了'

    fav = Favorite()
    fav.u_id = u_id
    fav.m_id = m_id

    db.session.add(fav)
    db.session.commit()
    return '添加fav 成功'


@fav_blue.route('/cancel_the_collection/')
def cancel_the_collection():
    """02取消收藏"""
    u_id = request.args.get('u_id')
    m_id = request.args.get('m_id')

    ex_fav = Favorite.query.filter(and_(Favorite.u_id == u_id,
                                        Favorite.m_id == m_id))

    # print(ex_fav.first())   #<Favorite 1>
    fav = ex_fav.first()
    db.session.delete(fav)
    db.session.commit()
    return '取消收藏成功'

~~~





#### ---templates\showstudent.html---

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>showstudent</title>
</head>
<body>
<ul>
    {% if students %}
        {% for stu in students %}
            <li>姓名:{{ stu.s_name }} 分数:{{ stu.s_score }}</li>
        {% endfor %}

    {% endif %}
</ul>

</body>
</html>
~~~



------

- `python manager.py db init`  创建一个alembic仓库
- `python manager.py db migrate`  生成迁移文件
- `python manager.py db upgrade`  将指定版本的迁移文件映射到数据库中
- `python manager.py runserver -r -h 0 -p 8000`  启动服务



**下面的是我已经初始化过仓库了；只不过删除了student表，来重新映射一下**



~~~
(fkpy3) kakaxi@ubuntu64:~/flaskcode/flaskday03$ python manager.py db init
/home/kakaxi/.virtualenvs/fkpy3/lib/python3.5/site-packages/flask_sqlalchemy/__init__.py:794: FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
  'SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and '
Error: Directory migrations already exists





(fkpy3) kakaxi@ubuntu64:~/flaskcode/flaskday03$ python manager.py db migrate
/home/kakaxi/.virtualenvs/fkpy3/lib/python3.5/site-packages/flask_sqlalchemy/__init__.py:794: FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
  'SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and '
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'student'
  Generating /home/kakaxi/flaskcode/flaskday03/migrations/versions/02dc5c45c422_.py ... done
  
  
  
  
  
(fkpy3) kakaxi@ubuntu64:~/flaskcode/flaskday03$ python manager.py db upgrade
/home/kakaxi/.virtualenvs/fkpy3/lib/python3.5/site-packages/flask_sqlalchemy/__init__.py:794: FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
  'SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and '
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade cb90fa29d773 -> 02dc5c45c422, empty message





(fkpy3) kakaxi@ubuntu64:~/flaskcode/flaskday03$ python manager.py runserver -r -h 0 -p 8000
/home/kakaxi/.virtualenvs/fkpy3/lib/python3.5/site-packages/flask_sqlalchemy/__init__.py:794: FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
  'SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and '
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0:8000/ (Press CTRL+C to quit)
 * Restarting with stat
/home/kakaxi/.virtualenvs/fkpy3/lib/python3.5/site-packages/flask_sqlalchemy/__init__.py:794: FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
  'SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and '
 * Debugger is active!
 * Debugger PIN: 182-666-561

~~~

