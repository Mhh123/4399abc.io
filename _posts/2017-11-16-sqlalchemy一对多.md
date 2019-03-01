```python

#注意看
#ForeignKey('department3.id')外键所在的表(student3)为多
#relationship('student3',backref='department3')所在的表(department3)backref是反向查询的意思
#department3-->student3为一对多

from connect2 import engine,Base
from sqlalchemy import Column,ForeignKey,Integer,String,Boolean,Text,Date,DateTime
from sqlalchemy.orm import sessionmaker,relationship
Session=sessionmaker(engine)
session=Session()
class department3(Base):
    __tablename__='department3'
    id=Column(Integer,primary_key=True,autoincrement=True)
    name=Column(String(20))
    student3=relationship('student3',backref='department3')
    def __repr__(self):
        return '<dep3(id="%s",name="%s")>'%(self.id,self.name)
class student3(Base):
    __tablename__='student3'
    s_id=Column(Integer,primary_key=True,autoincrement=True)
    s_name=Column(String(20))
    d_id=Column(Integer,ForeignKey('department3.id'))
    def __repr__(self):
        return '<student3(s_id="%s",s_name="%s",d_id="%s")>'%(self.s_id,self.s_name,self.d_id)
# Base.metadata.create_all()
ls=[department3(name='木叶忍者村'),department3(name='砂隐忍者村'),department3(name='云隐忍者村'),
    department3(name='雾隐忍者村'),department3(name='岩隐忍者村')]
# session.add_all(ls)
# session.commit()
my1=[student3(s_name='旗木卡卡西',d_id=1),student3(s_name='猿飞阿斯玛',d_id=1),student3(s_name='御手洗红豆',d_id=1)]
sy1=[student3(s_name='沙瀑之我爱罗 ',d_id=2),student3(s_name='风间勘九郎',d_id=2),student3(s_name='风间勘九郎',d_id=2)]
yr2=[student3(s_name='金角',d_id=3),student3(s_name='银角',d_id=3),student3(s_name='打不死',d_id=3)]
wy1=[student3(s_name='桃地再不斩',d_id=4),student3(s_name='鬼灯水月',d_id=4),student3(s_name='干柿鬼鲛',d_id=4)]
yr1=[student3(s_name='山椒鱼半藏',d_id=5),student3(s_name='君麻吕',d_id=5),student3(s_name='多由也',d_id=5)]
Als=my1+sy1+yr1+wy1+yr1
# session.add_all(Als)
# session.commit()
session.add_all(yr2)
session.commit()
```
