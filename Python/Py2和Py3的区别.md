### print 函数

print语句没有了，取而代之的是print()函数。 Python 2.6与Python 2.7部分地支持这种形式的print语法。  

### Unicode

Python 2 有 ASCII str() 类型，unicode() 是单独的，不是 byte 类型。  
现在， 在 Python 3，我们最终有了 Unicode (utf-8) 字符串，以及一个字节类：byte 和 bytearrays。  

python3.py
```python
中国 = 'china' 
print(中国) 
#china
```
### 除法运算
python2.py
```py
>>> 1 / 2
0
>>> 1.0 / 2.0
0.5
>>> -1 // 2
-1
```

python3.py
```py
>>> 1/2
0.5
>>> -1 // 2
-1
```

### 异常

捕获异常的语法由 except exc, var 改为 except exc as var。  

### xrange 
取消了xrange，range代替了xrange。  

### 不等运算符
Python 2.x中不等于有两种写法 != 和 <>  
Python 3.x中去掉了<>, 只有!=一种写法，还好，我从来没有使用<>的习惯  
 
### 去掉了repr表达式\`\`
Python 2.x 中反引号\`\`相当于repr函数的作用  
Python 3.x 中去掉了\`\`这种写法，只允许使用repr函数  
 
### 多个模块被改名（根据PEP8）

- 旧的名字	新的名字
- _winreg	winreg
- ConfigParser	configparser
- copy_reg	copyreg
- Queue	queue
- SocketServer	socketserver
- repr	reprlib

StringIO模块现在被合并到新的io模组内。 new, md5, gopherlib等模块被删除。 Python 2.6已经支援新的io模组。  
httplib, BaseHTTPServer, CGIHTTPServer, SimpleHTTPServer, Cookie, cookielib被合并到http包内。  
取消了exec语句，只剩下exec()函数。 Python 2.6已经支援exec()函数。  

### 数据类型
1. Py3.X去除了long类型，现在只有一种整型——int，但它的行为就像2.X版本的long  
2. 新增了bytes类型，对应于2.X版本的八位串，定义一个bytes字面量的方法如下：
python3.py
```py
>>> b = b'china' 
>>> type(b) 
<type 'bytes'>
# str对象和bytes对象可以使用.encode() (str -> bytes) or .decode() (bytes -> str)方法相互转化。
>>> s = b.decode() 
>>> s 
'china' 
>>> b1 = s.encode() 
>>> b1 
b'china' 
```

### 迭代器
在 Python2 中很多返回列表对象的内置函数和方法在 Python 3 都改成了返回类似于迭代器的对象，因为迭代器的惰性加载特性使得操作大数据更有效率。  
字典对象的 dict.keys()、dict.values() 方法都不再返回列表，而是以一个类似迭代器的 "view" 对象返回。高阶函数 map、filter、zip 返回的也都不是列表对象了。Python2的迭代器必须实现 next 方法，而 Python3 改成了`__next__ `  
### True和False
True 和 False 变为两个关键字，永远指向两个固定的对象，不允许再被重新赋值。    
```py
try:
	True='asdf'
	print(True)
except BaseException as e:
	print(e)
# asdf //python2
# SyntaxError: can't assign to keyword  //python3
```
### 函数
1. python2可以使用file和open打开文件，python3只能使用open。

2. 在python2.x中raw_input()和input( )，两个函数都存在，其中区别为：
- raw_input()---将所有输入作为字符串看待，返回字符串类型
- input()-----只能接收"数字"的输入，在对待纯数字输入时具有自己的特性，它返回所输入的数字的类型（int, float ）
在python3.x中raw_input()和input( )进行了整合，去除了raw_input()，仅保留了input()函数，其接收任意任性输入，将所有输入默认为字符串处理，并返回字符串类型。  

3. map、filter 和 reduce
python2
```py
>>> map
<built-in function map>
>>> filter
<built-in function filter>
# 输出列表
>>> map(lambda x:x *2, [1,2,3])
[2, 4, 6]
>>> filter(lambda x:x %2 ==0,range(10))
[0, 2, 4, 6, 8]
>>>
```
python3
```py
>>> map
<class 'map'>
>>> map(print,[1,2,3])
<map object at 0x10d8bd400>
>>> filter
<class 'filter'>
>>> filter(lambda x:x % 2 == 0, range(10))
<filter object at 0x10d8bd3c8>
>>> f =filter(lambda x:x %2 ==0, range(10))
>>> next(f)
0
>>> next(f)
2
>>> next(f)
4
>>> next(f)
6
```
对于比较高端的 reduce 函数，它在 Python 3.x 中已经不属于 built-in 了，被挪到 functools 模块当中。