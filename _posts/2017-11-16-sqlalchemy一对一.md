```python


#注意看
#ForeignKey('member.id')外键所在的表(Me_detail)为多
#relationship('Me_detail',uselist=False)所在的表(Member)如果加上uselist=False则为一
#Member-->Me_detail为一对一
#为什么是一对一呢？不应该是一对多吗？
#因为限制了关联列,所以本来是多选的,现在变成了单选，就变成了一对一
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
from sqlalchemy.orm import relationship
class Member(Base):
    __tablename__='member'
    id=Column(Integer,primary_key=True,autoincrement=True)
    name=Column(String(20))

    detail=relationship('Me_detail',uselist=False)
    def __repr__(self):
        return '<Member(id="%s",name="%s")>'%(self.id,self.name)

class Me_detail(Base):
    __tablename__='me_detail'
    id=Column(Integer,primary_key=True,autoincrement=True)
    m_id=Column(Integer,ForeignKey('member.id'),unique=True)
    m_name=Column(String(20))

    member=relationship('Member')
    def __repr__(self):
        return '<Me_detail(id="%s",m_id="%s",m_name="%s")>'%(self.id,self.m_id,self.m_name)

Base.metadata.create_all()
from sqlalchemy.orm import sessionmaker
Session=sessionmaker(engine)
session=Session()
# ls1=[Member(name='鸣人'),Member(name='佐助'),Member(name='小樱')]
# session.add_all(ls1)
# session.commit()
# rs=session.query(Member).all()
# print(rs)
# ls2=[Me_detail(m_id=1,m_name='漩涡鸣人'),Me_detail(m_id=2,m_name='宇智波佐助'),Me_detail(m_id=3,m_name='春野樱')]
# session.add_all(ls2)
# session.commit()
# Mt=session.query(Me_detail).all()
# print(Mt)
# L1=session.query(Member).all()
# [print(L1[i],L1[i].detail) for i in range(len(L1))]
# L2=session.query(Me_detail).all()
# [print(L2[i],L2[i].member) for i in range(len(L2))]


```
