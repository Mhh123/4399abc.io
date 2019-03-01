## flask-restful



#### 安装

​	`pip install flask-restful`





#### 配置

**---app\__init__.py---**

~~~python
from app.urls import init_api

def create_app(param):
	init_api(app)
	return app
~~~

**---app\urls.py---**

~~~python
from flask_restful import Api

from .apis import DemoResource
	
api = Api()

def init_api(app):
    api.init_app(app)
    
 api.add_resource(DemoResource, '/demos/', '/hello/', endpoint='test')
~~~



## 项目结构

~~~
flaskday05
├── app
│   ├── apis.py
│   ├── ext.py
│   ├── __init__.py
│   ├── models.py
│   ├── __pycache__
│   │   ├── apis.cpython-35.pyc
│   │   ├── ext.cpython-35.pyc
│   │   ├── __init__.cpython-35.pyc
│   │   ├── models.cpython-35.pyc
│   │   ├── settings.cpython-35.pyc
│   │   └── urls.cpython-35.pyc
│   ├── settings.py
│   ├── static
│   ├── templates
│   │   └── show_stus.html
│   ├── urls.py
│   └── views
│       ├── __init__.py
│       ├── my_t5.py
│       ├── person_view.py
│       ├── __pycache__
│       │   ├── __init__.cpython-35.pyc
│       │   ├── my_t5.cpython-35.pyc
│       │   ├── person_view.cpython-35.pyc
│       │   └── student_view.cpython-35.pyc
│       └── student_view.py
├── __init__.py
├── manager.py
└── migrations
    ├── alembic.ini
    ├── env.py
    ├── __pycache__
    │   └── env.cpython-35.pyc
    ├── README
    ├── script.py.mako
    └── versions
        ├── 344ead93663d_.py
        ├── 98b80124c560_.py
        ├── cbc23b04c38b_.py
        └── __pycache__
            ├── 344ead93663d_.cpython-35.pyc
            ├── 98b80124c560_.cpython-35.pyc
            └── cbc23b04c38b_.cpython-35.pyc

~~~



#### 例子1:

**---views\student_view.py---**

野蛮的的result风格（我自己起的名字）

~~~python
import random

from flask import Blueprint, render_template, abort, request, jsonify

from app.models import Student

from app.ext import db

from app.ext import cache

stu_blue = Blueprint('stu_blue',__name__)

@stu_blue.route('/add_students/')
def add_students():
    students = []
    for i in range(10):
        num = random.randrange(80)
        stu = Student()
        stu.s_name = 'wudi %d' %  num
        stu.s_age = 19
        stu.s_score = num + 20
        students.append(stu)

    db.session.add_all(students)
    db.session.commit()
    return '批量添加学生ok'

@stu_blue.route('/get_all_student/')
def get_all_student():
    students = Student.query.all()
    return render_template('show_stus.html',students=students)

@stu_blue.before_request
def handler_before():   #中间件

    count = int(cache.get('count') or 1)

    if count >= 10:
        abort(404)

    count += 1
    cache.set('count',count,timeout=10)

#路径直接使用名词，而且使用复数
@stu_blue.route('/students/',methods = ['GET','POST','PUT','PATCH','DELETE'])
def my_students():
    if request.method == 'GET':
        students = Student.query.all()

        dict = {'msg':'获取成功','status_code':200}

        #dict['data'] = students
        #这样写会报错not iterable，
        # 解决方案，object转字典，使用to_dict

        students_dict = []
        for stu in students:
            students_dict.append(stu.to_dict())

        dict['data'] = students_dict


        return jsonify(dict)
    elif request.method == 'POST':
        dict = {'msg': '新建成功', 'status_code': 201}

        name = request.form.get('name')
        age = request.form.get('age')
        score = request.form.get('score')

        if not name or not age or not score:
            dict['msg'] = '参数不对'
            dict['status_code'] = 400
            return jsonify(dict)

        stu = Student()
        stu.s_name = name
        stu.s_age = age
        stu.s_score = score

        db.session.add(stu)
        db.session.commit()

        dict['data'] = stu.to_dict()


        return jsonify(dict)

    elif request.method == 'PUT':   #更新某一个数据的全部属性
        s_id = request.form.get('id')
        s_name = request.form.get('name')
        s_age = request.form.get('age')
        s_score = request.form.get('score')


        stu = Student.query.get(s_id)

        if stu:
            stu.s_name = s_name
            stu.s_age = s_age
            stu.s_score = s_score
            try:
                db.session.add(stu)
                db.session.commit()
            except Exception as e:
                return jsonify({'msg':'修改失败','status_code':404})
        else:
            return jsonify({'msg':'没有找到数据','status_code':404})


        return jsonify({'msg': '更新成功', 'status_code': 201})
    elif request.method == 'PATCH':
        return jsonify({'msg': '更新成功', 'status_code': 201})
    elif request.method == 'DELETE':
        id = int(request.form.get('id'))

        stu = Student.query.get(id)

        if not stu:
            return jsonify({'msg':'没有找到数据','status_code':404})

        try:
            db.session.delete(stu)
            db.session.commit()
        except Exception as e:
            return jsonify({'msg':'删除失败','status_code':400})


        return jsonify({'msg': '删除成功', 'status_code': 204})
    else:
        abort(405)
~~~

#### 例子2:

**---app\apis.py---**

正统的restful风格(安装了flask-restful)

~~~python
from flask import request
from flask_restful import Resource, fields, marshal_with

from app.models import Person, Address, Movie

from app.ext import db

# 下面所有的请求都是使用Postman模拟的



demo_fields = {
    'msg': fields.String,
    'status_code': fields.Integer,
    'private_key': fields.String(attribute='password'),  # 相当于password取别名
    'my_default': fields.String(default='我是defaultvalue'),
    'uri': fields.Url('test', absolute=True)  # Url('todo_resource')
}


class DemoResource(Resource):
    """01 restful 风格 + 装饰器"""

    @marshal_with(demo_fields)
    def get(self):
        # 如果 'private_key':123后显示private_key :null;
        # 因为 fields.String(attribute='password')
        # 所以 要写 'passowrd':123 页面才能接收到
        return {'msg': 'hello world', 'status_code': 200, 'password': 123}

    # 访问http://mhh4399.com:8000/demos/ 返回
    # {
    #     "my_default": "我是defaultvalue",
    #     "uri": "http://mhh4399.com:8000/demos/",
    #     "status_code": 200,
    #     "private_key": "123",
    #     "msg": "hello world"
    # }

    def post(self):
        return {'msg': '我是post'}


# 对addr进行格式化
addr_fields = {
    'a_province': fields.String,
    'a_shortname': fields.String
}
data_fields = {
    'msg': fields.String,
    'status_code': fields.Integer,
    'data': fields.Nested(addr_fields)  # Nested相当于继承addr_fields
}


class AddrResource(Resource):
    """ 02 通过继承和装饰器的组合使用 让 data 返回一个字典"""

    @marshal_with(data_fields)
    def get(self, a_id):
        addr = Address.query.get(a_id)

        return {'msg': 'get  ok', 'status_code': 200, 'data': addr}

        #  访问 http://mhh4399.com:8000/address/1/
        # {
        #     "status_code": 200,
        #     "data": {
        #         "a_shortname": "台",
        #         "a_province": "台湾"
        #     },
        #     "msg": "get  ok"
        # }


person_fields = {
    'p_name': fields.String,
    'p_province': fields.String
}

result_fields = {
    'msg': fields.String,
    'status_code': fields.Integer,
    'data': fields.List(fields.Nested(person_fields))
}


class PersonResource(Resource):
    """ 03  获取所有的person对象 fields.List """

    @marshal_with(result_fields, envelope='test')  # envelope  把所有的输出再包裹起来，起个名(就是为了好看)
    def get(self):
        persons = Person.query.all()

        # 笨方法

        # per_dicts = []
        # for per in persons:
        #     per_dicts.append(per.to_dict())

        # return {'msg':'获取成功','status_code':200,'dada':per_dicts}

        # 聪明方法
        return {'msg': '获取成功', 'status_code': 200, 'data': persons}

        # 扩展，marshal方法直接调用，替代装饰器
        # return marshal({'msg': '获取成功', 'status_code': 200,result_fields)}

        # 访问 http://mhh4399.com:8000/persons/ 返回

        # {
        #     "test": {
        #         "msg": "获取成功",
        #         "data": [
        #             {
        #                 "p_name": "张三",
        #                 "p_province": "台湾"
        #             },
        #             {
        #                 "p_name": "李四",
        #                 "p_province": "上海"
        #             },
        #             {
        #                 "p_name": "王五",
        #                 "p_province": "北京"
        #             },
        #             {
        #                 "p_name": "赵六",
        #                 "p_province": "香港"
        #             }
        #         ],
        #         "status_code": 200
        #     }
        # }


movie_fields = {
    'name': fields.String(attribute='m_name')
}

movie_result = {
    'msg': fields.String,
    'status_code': fields.Integer,
    'data': fields.Nested(movie_fields)
}


class MovieResource(Resource):
    @marshal_with(movie_result)
    def get(self):
        m_id = request.args.get('m_id')
        movie = Movie.query.get(m_id)

        return {'msg': 'ok', 'status_code': 200, 'data': movie}

    # 访问 http: // mhh4399.com: 8000 / movies /?m_id = 1  返回
    # {
    #     "status_code": 200,
    #     "msg": "ok",
    #     "data": {
    #         "name": "钢铁侠"
    #     }
    # }

    @marshal_with(movie_result)
    def post(self):
        m_name = request.form.get('m_name')
        movie = Movie()
        movie.m_name = m_name

        db.session.add(movie)
        db.session.commit()

        return {'msg': 'ok', 'status_code': 200, 'data': movie}


        # 访问 http://mhh4399.com:8000/movies/  返回

        # body:
        #     form-data
        #     Key      Value
        #     m_name   无人区

        # {
        #     "status_code": 200,
        #     "msg": "ok",
        #     "data": {
        #         "name": "无人区"
        #     }
        # }

~~~



------

###### 下面的可看可不看



###### **---ext.py---**

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

###### **---settings.py---**

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
        'databases': 'my_flask05'  # 数据库名
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

###### **---models.py---**

~~~python
from app.ext import db
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship


class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    s_name = db.Column(db.String(32), nullable=False)
    s_age = db.Column(db.Integer, nullable=False, default=18)
    s_score = db.Column(db.Float, nullable=False, default=0)

    def to_dict(self):
        return {'s_name':self.s_name,'s_age':self.s_age,'s_score':self.s_score}

class Address(db.Model):
    # __table__name = 'xxx'
    id = Column(Integer, primary_key=True, autoincrement=True)
    a_province = Column(String(16), nullable=False, unique=True)
    a_shortname = Column(String(16), nullable=False)

    a_person = relationship('Person', backref='addr')


class Person(db.Model):
    id = Column(Integer, primary_key=True, autoincrement=True)
    p_name = Column(String(32), unique=True, nullable=False)
    p_province = Column(String(16),ForeignKey(Address.a_province))

    def to_dict(self):
        return {'s_name':self.s_name,'s_age':self.s_age,'s_score':self.s_score}


class User(db.Model):
    id = Column(Integer, primary_key=True, autoincrement=True)
    u_name = Column(String(128), unique=True, nullable=False)

class Movie(db.Model):
    id = Column(Integer, primary_key=True, autoincrement=True)
    m_name = Column(String(32), unique=True, nullable=False)

class Favorite(db.Model):
    id = Column(Integer, primary_key=True, autoincrement=True)
    u_id = Column(Integer,ForeignKey(User.id))
    m_id = Column(Integer,ForeignKey(Movie.id))
~~~

###### **---views\__init__.py---**

~~~python
from app.ext import init_ext
from app.settings import env

from app.views.student_view import stu_blue

from app.urls import init_api

from .views.person_view import per_blue
from .views.my_t5 import t5_blue

from flask import Flask


def create_app(param):
    app = Flask(__name__)
    app.config.from_object(env.get(param))  # 建议把配置项放在前面；避免有时候放后面，配置项不起效
    # app必须要注册一个蓝图才能使用
    app.register_blueprint(blueprint=t5_blue)
    app.register_blueprint(blueprint=stu_blue)
    app.register_blueprint(blueprint=per_blue)


    init_ext(app)
    init_api(app)
    return app

~~~

###### **---manager.py---**

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