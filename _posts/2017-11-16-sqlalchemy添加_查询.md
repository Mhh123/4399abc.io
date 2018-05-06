```python


#注意join
# SQL JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。
# 最常见的 JOIN 类型：SQL INNER JOIN（简单的 JOIN）。 SQL INNER JOIN 从多个表中返回满足 JOIN 条件的所有行。
# 不同的 SQL JOIN
# 在我们继续讲解实例之前，我们先列出您可以使用的不同的 SQL JOIN 类型：
# INNER JOIN：如果表中有至少一个匹配，则返回行
# LEFT JOIN：即使右表中没有匹配，也从左表返回所有的行
# RIGHT JOIN：即使左表中没有匹配，也从右表返回所有的行
# FULL JOIN：只要其中一个表中存在匹配，则返回行

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
from sqlalchemy import Column,ForeignKey,Integer,String
from sqlalchemy.orm import relationship,sessionmaker
Session=sessionmaker(engine)
session=Session()
class User(Base):
    __tablename__='user'
    id=Column(Integer,primary_key=True,autoincrement=True)
    name=Column(String(20))

    skill=relationship('Skill',secondary='us_skill')
    def __repr__(self):
        return '<User(id="%s")>'%(self.name)

class Skill(Base):
    __tablename__='skill'
    s_id=Column(Integer,primary_key=True,autoincrement=True)
    s_name=Column(String(20))

    user=relationship('User',secondary='us_skill')
    def __repr__(self):
        return '<Skill(s_name="%s")>'%(self.s_name)

class Us_skill(Base):
    __tablename__='us_skill'
    us_id=Column(Integer,ForeignKey('user.id'),primary_key=True)
    sk_id=Column(Integer,ForeignKey('skill.s_id'),primary_key=True)

    def __repr__(self):
        return '<Us_skill(u_id="%s",s_id="%s")>'%(self.us_id,self.sk_id)

Base.metadata.create_all()
u1=User(name='旗木卡卡西')
u2=User(name='宇智波带土')
u3=User(name='纳凉琳琳')
s1=Skill(s_name='千鸟雷切')
s2=Skill(s_name='写轮眼')
s3=Skill(s_name='治疗术')
s4=Skill(s_name='火遁术')
# u1.skill.extend([s1,s2])
# u2.skill.append(s2)
# u3.skill.append(s3)

#注意添加数据,只要添加原始数据即可，被关联的数据会自动添加,没有关联到的原始数据,需要添加
#比如:在下面例子中,u1,u2,u3原始数据,s1,s2,s3是被关联的所以不用再添加，
# 但是s4是没有被任何关联的，所以需要单独添加
# session.add_all([u1,u2,u3,s4])
# session.commit()

# l1_a=session.query(User).filter(User.name=='旗木卡卡西').first()
# print(l1_a.skill)
# l1_b=session.query(User).filter(User.name=='宇智波带土').first()
# print(l1_b.skill)

# l2_a=session.query(Skill).filter(Skill.s_name=='写轮眼').first()
# print(l2_a.user)
# rs=session.query(User.name,Skill.s_name).join(Us_skill).filter(
#     User.id==Us_skill.us_id,Skill.s_id==Us_skill.sk_id).all()
# print(rs)

# rs1=session.query(User.name,Skill.s_name).join(Skill,User.id==Skill.s_id).all()
# print(rs1)
# l2=Skill(s_name='真的垃圾技能')
# session.add(l2)
# session.commit()
#少对多 (join);多对少 (outerjoin);
# rs2=session.query(User.name,Skill.s_name).outerjoin(Skill,Skill.s_id == User.id).all()
# print(rs2)
#sqlalchemy中没有右链接，不过可以调换表的位置实现
# rs3=session.query(Skill,User).outerjoin(User,User.id==Skill.s_id).all()
# print(rs3)
```

