

functools模块

@functools.wraps的作用:

​	把原始函数的`__name__`等属性复制到`wrapper()`函数中

[详情]: https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318435599930270c0381a3b44db991cd6d858064ac0000	"廖雪峰官网"



```python
>>> def log(func):
...     def wrapper(*args,**kwargs):
...             print("begin Call")
...             func(*args,**kwargs)
...             print("end Call")
...     return wrapper
... 
>>> @log
... def func1():
...     print("running func1")
... 
>>> func1.__name__
'wrapper'
>>> func1()
begin Call
running func1
end Call

```

因为返回的那个`wrapper()`函数名字就是`'wrapper'`，所以，需要把原始函数的`__name__`等属性复制到`wrapper()`函数中，否则，有些依赖函数签名的代码执行就会出错。

不需要编写`wrapper.__name__ = func.__name__`这样的代码，Python内置的`functools.wraps`就是干这个事的，所以，一个完整的decorator的写法如下：

```
>>> import functools
>>> def log(func):
...     @functools.wraps(func)
...     def wrapper(*args,**kwargs):
...             print("begin Call")
...             ret = func(*args,**kwargs)
...             print("end Call")
...             return ret
...     return wrapper
... 
>>> @log
... def func2():
...     print("running func2")
... 
>>> func2.__name__
'func2'
>>> func2()
begin Call
running func2
end Call
>>> 

```

