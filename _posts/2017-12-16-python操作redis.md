```python
import redis
#默认数据库0~15,先使用第0个数据库
#在redis里面可以通过select num 来选择使用那个数据库,在python里面可以设置db=num
r=redis.Redis(host='localhost',port=8888,db=0)
'''
    Provides backwards compatibility with older versions of redis-py that
    changed arguments to some commands to be more Pythonic, sane, or by
    accident.
'''
# r.set('obj','4399abc')
# print(r.get('obj'))

#解码输出
# r.set('name','宇智波佐助')
# print(r.get('name').decode('utf8'))

#设置多个key value 注意传入的参数必须是dict arg
#MSET requires **kwargs or a single dict arg
# r.mset({'a':1,'b':2})

#append添加
# r.append('name','-漩涡鸣人')
# print(r.get('name').decode('utf8'))
# r.delete('obj')
#将 key 所储存的值加上或者减去给定的增量值
# print(r.get('count'))
# r.incr('count',50)
# print(r.get('count'))
# r.decr('count',100)
# print(r.get('count'))

#列表
# r.lpush('list',1,2,3)
# print(r.lrange('list',0,-1))
#哈希
# r.hset('hash','name','json')
# print(r.hgetall('hash'))
# r.hset('hash1','name1','nil')
# print(r.hgetall('hash1'))
#集合(特点：唯一，无序)
# r.sadd('set',1,2,3,4,4,3,2,1)
# print(r.smembers('set'))
# r.srem('set',4,3) #可以一处一个或多个成员
# print(r.smembers('set'))
# r.sadd('set',1,2,3,4,4,3,2,1)
# print(r.spop('set'),r.smembers('set'),sep='\n')#pop是随机的，并且返回自身
#有序集合 zset(唯一，有序)是通过一个分数来实现排序的.
# r.zadd('zset','a',1,'b',2)
# print(r.zrange('zset',0,-1))
# print(r.zrange('zset',0,-1,withscores=True))#显示分数
# r.zadd('zset','a',1,'b',2)
# r.zrem('zset','a')
# print(r.zrange('zset',0,-1,withscores=True))
# r.zadd('zset','a',1,'b',2)
# print(r.zrank('zset','b'))#返回成员的索引，注意是索引，不是分数




```

