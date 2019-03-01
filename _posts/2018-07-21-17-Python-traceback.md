## Python  traceback



python中用于处理异常栈的模块是traceback模块，它提供了print_exception、format_exception等输出异常栈等常用的工具函数。



- Changed in version 3.5: The *etype* argument is ignored and inferred from the type of *value*.

  ~~~
  traceback.print_exc`(limit=None, file=None, chain=True)

  This is a shorthand for print_exception(*sys.exc_info(), limit, file, chain).

  ~~~

  ​

- Changed in version 3.5: The *etype* argument is ignored and inferred from the type of *value*.

  ~~~	
  traceback.format_exc`(limit=None, chain=True)

  This is like print_exc(limit) but returns a string instead of printing to a file.

  ~~~



```
>>> def func(a,b):
	return a/b

>>> import sys,traceback
>>> try:
	func(1,0)
except Exception:
	print("*** exc")
	traceback.print_exc(file=sys.stdout)

	
*** exc
Traceback (most recent call last):
  File "<pyshell#75>", line 2, in <module>
  File "<pyshell#68>", line 2, in func
ZeroDivisionError: division by zero
>>> 

```

其实traceback.print_exc()函数只是traceback.print_exception()函数的一个简写形式，而它们获取异常相关的数据都是通过sys.exc_info()函数得到的。

```python
>> import sys,traceback
>>> def func(a,b):
	return a/b

>>> try:
	func(1,0)
except Exception:
	print("*** print_exception()")
	exc_type, exc_value, exc_tb = sys.exc_info()
	print("exc_type --->:", exc_type)
	print("exc_value --->:", exc_value)
	print("exc_tb --->:", exc_tb)
	traceback.print_exception(exc_type, exc_value, exc_tb)

	
*** print_exception()
exc_type --->: <class 'ZeroDivisionError'>
exc_value --->: division by zero
exc_tb --->: <traceback object at 0x0000000003312E48>
Traceback (most recent call last):
  File "<pyshell#93>", line 2, in <module>
  File "<pyshell#83>", line 2, in func
ZeroDivisionError: division by zero
```



sys.exc_info()返回的值是一个元组，其中第一个元素，exc_type是异常的对象类型，exc_value是异常的值，exc_tb是一个traceback对象，对象中包含出错的行数、位置等数据。然后通过print_exception函数对这些异常数据进行整理输出。

traceback模块提供了extract_tb函数来更加详细的解释traceback对象所包含的数据：

```python
>>> import sys,traceback
>>> def func(a,b):
	return a/b

>>> try:
	func(1,0)
except Exception:
	_, _, exc_tb = sys.exc_info()
	for filename, linenum, funcname, source in traceback.extract_tb(exc_tb):
		print("%-23s:%s '%s' in %s()"%(filename,linenum,source,funcname))

		
<pyshell#100>          :2 '' in <module>()
<pyshell#83>           :2 '' in func()
>>> 
```





------

下面是官网例子 

https://docs.python.org/3/library/traceback.html?highlight=traceback#module-traceback

### Traceback Examples



This simple example implements a basic read-eval-print loop, similar to (but less useful than) the standard Python interactive interpreter loop. 

~~~python
>>> import sys,traceback
>>> def run_user_code(envdir):
	source = input(">>> ")
	try:
		exec(source,envdir)
	except Exception:
		print("Exception in user code:")
		print("-"*60)
		traceback.print_exc(file=sys.stdout)
		print("-"*60)

		
>>> envdir = {}
>>> while True:
	run_user_code(envdir)
	
	
>>> 
>>> abc
Exception in user code:
------------------------------------------------------------
Traceback (most recent call last):
  File "<pyshell#24>", line 4, in run_user_code
  File "<string>", line 1, in <module>
NameError: name 'abc' is not defined
------------------------------------------------------------
>>> 1/0
Exception in user code:
------------------------------------------------------------
Traceback (most recent call last):
  File "<pyshell#24>", line 4, in run_user_code
  File "<string>", line 1, in <module>
ZeroDivisionError: division by zero
------------------------------------------------------------
>>> 
~~~



The following example demonstrates the different ways to print and format the exception and traceback:

~~~python
>>> import sys,traceback
>>> 
>>> def lumberjack():
	bright_side_of_death()
	
>>> def bright_side_of_death():
	return tuple()[0]

>>> try:
	lumberjack()
except IndexError:
	exc_type,exc_value,exc_traceback = sys.exc_info()
	print("*** print_tb:")
	traceback.print_tb(exc_traceback,limit=1,file=sys.stdout)
	print("*** print_exception:")
	# exc_type below is ignored on 3.5 and later
	traceback.print_exception(exc_type,exc_value,exc_traceback,
				  limit=2,file=sys.stdout)
	print("*** print_exc:")
	traceback.print_exc(limit=2,file=sys.stdout)
	print("*** format_exc,first and last line:")
	formatted_lines = traceback.format_exc().splitlines()
	print(formatted_lines[0])
	print(formatted_lines[-1])
	print("*** format_exception:")
	#exc_type below is ignored on 3.5 and later
	print(repr(traceback.format_exception(exc_type,exc_value,
					      exc_traceback)))
	print("*** extract_tb:")
	print(repr(traceback.extract_tb(exc_traceback)))
	print("*** format_tb:")
	print(repr(traceback.format_tb(exc_traceback)))
	print("*** tb_lineno:",exc_traceback.tb_lineno)

	
*** print_tb:
  File "<pyshell#65>", line 2, in <module>
*** print_exception:
Traceback (most recent call last):
  File "<pyshell#65>", line 2, in <module>
  File "<pyshell#33>", line 2, in lumberjack
IndexError: tuple index out of range
*** print_exc:
Traceback (most recent call last):
  File "<pyshell#65>", line 2, in <module>
  File "<pyshell#33>", line 2, in lumberjack
IndexError: tuple index out of range
*** format_exc,first and last line:
Traceback (most recent call last):
IndexError: tuple index out of range
*** format_exception:
['Traceback (most recent call last):\n', '  File "<pyshell#65>", line 2, in <module>\n', '  File "<pyshell#33>", line 2, in lumberjack\n', '  File "<pyshell#37>", line 2, in bright_side_of_death\n', 'IndexError: tuple index out of range\n']
*** extract_tb:
[<FrameSummary file <pyshell#65>, line 2 in <module>>, <FrameSummary file <pyshell#33>, line 2 in lumberjack>, <FrameSummary file <pyshell#37>, line 2 in bright_side_of_death>]
*** format_tb:
['  File "<pyshell#65>", line 2, in <module>\n', '  File "<pyshell#33>", line 2, in lumberjack\n', '  File "<pyshell#37>", line 2, in bright_side_of_death\n']
*** tb_lineno: 2
>>> 
~~~



转博客: https://blog.csdn.net/lengxingxing_/article/details/56317838