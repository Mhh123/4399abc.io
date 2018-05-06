```python
from sqlalchemy import create_engine
Username='mhh123'
Password='mhh123'
Host='192.168.40.128'
Port=3306
Database='mydb'
DB_URI='mysql+pymysql://{}:{}@{}:{}/{}?charset=utf8'.format(Username,Password,Host,Port,Database)
engine=create_engine(DB_URI)
from sqlalchemy.ext.declarative import declarative_base
Base=declarative_base(engine)
from sqlalchemy import Column,ForeignKey,Integer,String,Boolean
from sqlalchemy.orm import relationship
class User(Base):
    __tablename__='user'
    id=Column(Integer,primary_key=True,autoincrement=True)
    u_id=Column(Integer,unique=True,nullable=False)
    u_name=Column(String(20))

    skill=relationship('Skill',secondary='us_skill')
    def __repr__(self):
        return '<User(id="%s",u_id="%s",u_name="%s")>'%(self.id,self.u_id,self.u_name)

class Skill(Base):
    __tablename__='skill'
    id=Column(Integer,primary_key=True,autoincrement=True)
    s_id=Column(Integer,nullable=False,unique=True)
    s_name=Column(String(20))

    user=relationship('User',secondary='us_skill')
    def __repr__(self):
        return '<Skill(id="%s",s_id="%s",s_name="%s")>'%(self.id,self.s_id,self.s_name)

class Us_skill(Base):
    __tablename__='us_skill'
    us_id=Column(Integer,ForeignKey('user.u_id'),primary_key=True)
    sk_id=Column(Integer,ForeignKey('skill.s_id'),primary_key=True)

Base.metadata.create_all()
from sqlalchemy.orm import sessionmaker
Session=sessionmaker(engine)
session=Session()
L1=[User(u_id=1,u_name='漩涡鸣人'),User(u_id=2,u_name='宇智波佐助'),User(u_id=3,u_name='春野樱')]
L2=[Skill(s_id=1,s_name='分身之术'),Skill(s_id=2,s_name='色诱之术'),Skill(s_id=3,s_name='通灵术'),
    Skill(s_id=4,s_name='螺旋丸'),Skill(s_id=5,s_name='火遁术'),Skill(s_id=6,s_name='千鸟术'),
    Skill(s_id=7,s_name='樱花吹雪之术'),Skill(s_id=8,s_name='忍法创造再生')]
L1[0].skill=L2[0:3]
# L1[0].skill.append(L2[3])
# session.add_all(L1)
# session.add_all(L2)
#注意session.add_all([L1,L2])这样写会报错,最好先L=L1+L2(将列表合并)
Als=L1+L2
#初次创建
# session.add_all(Als)

#第一次添加，为鸣人加技能
# session.add(Us_skill(us_id=1,sk_id=4))
# session.commit()
#二次添加，为佐助添加技能
# L1_l2=[Us_skill(us_id=2,sk_id=5),Us_skill(us_id=2,sk_id=6)]
# session.add_all(L1_l2)
# session.commit()
#三次添加,为小樱添加技能
# L1_l3=[Us_skill(us_id=3,sk_id=7),Us_skill(us_id=3,sk_id=8)]
# session.add_all(L1_l3)
# session.commit()
#查询
L1_1=session.query(User).filter(User.u_name=='漩涡鸣人').first()

L1_2=session.query(User).filter(User.u_name=='宇智波佐助').first()

L1_3=session.query(User).filter(User.u_name=='春野樱').first()
print(L1_1.skill,L1_2.skill,L1_3.skill,sep='\n')


```

