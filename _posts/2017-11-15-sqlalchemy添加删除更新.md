```python
from connect1 import engine,Base
from sqlalchemy import Column,Integer,String,Boolean,Text,Date,DateTime
from datetime import date,datetime
class xs1(Base):
    __tablename__='xd1'
    id=Column(Integer,primary_key=True,autoincrement=True)
    name=Column(String(20))
    def __repr__(self):
        return '<xs1(id="%s",name="%s")>'%(self.id,self.name)
# Base.metadata.create_all()
from sqlalchemy.orm import sessionmaker
Session=sessionmaker(engine)
session=Session()
#添加
# ls1=[xs1(name='自来也'),xs1(name='纲手'),xs1(name='自来也')]
# session.add_all(ls1)
# session.commit()
#更新
# a=session.query(xs1).filter(xs1.name=='纲手').all()[0]
# a.name='纲手大人'
#添加
# session.commit()
# session.add(xs1(name='猿飞日斩'))
# session.commit()
#更新
# session.query(xs1).filter(xs1.name=='猿飞日斩').update({xs1.id:11})
# session.commit()
#删除
# y=session.query(xs1).filter(xs1.id==5).first()
# session.delete(y)
# session.commit()
# ls2=[xs1(name='漩涡鸣人'),xs1(name='宇智波佐助'),xs1(name='春野樱')]
# session.add_all(ls2)
# session.commit()
# ls3=[xs1(name='日向宁次'),xs1(name='李洛克'),xs1(name='天天')]
# session.add_all(ls3)
# session.commit()
# ls4=[xs1(name='山中井野'),xs1(name='奈良鹿丸'),xs1(name='秋道丁次')]
# session.add_all(ls4)
# session.commit()
# x=session.query(xs1).filter(xs1.name!=None).all()
# [print(i) for i in x]
# x,y=session.query(xs1).filter(xs1.id<3).all(),session.query(xs1).filter(xs1.id>3,xs1.id<9).all()
# m,n=session.query(xs1).filter(xs1.id>=9,xs1.id<=11).all(),session.query(xs1).filter(xs1.id>=12,xs1.id<=14).all()
# print(x,y,m,n,sep='\n')

```

