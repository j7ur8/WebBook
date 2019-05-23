python2中可用使用input执行命令
```python
input(__import__('os').system('dir'))
```

一些任意代码执行以及文件读取的函数

代码执行
```python
import os
os.system('ipconfig')

exec('__import__("os").system("ipconfig")')
eval('__import__("os").system("ipconfig")')

import timeit
timeit.timeit("__import__('os').system('ipconfig')",number=1)
import platform
platform.popen('ipconfig').read()
import subprocess
subprocess.Popen('ipconfig', shell=True, stdout=subprocess.PIPE,stderr=subprocess.STDOUT).stdout.read()

```

文件读取
```python
file('/etc/passwd').read()  #python2
open('/etc/passwd').read()
import codecs
codecs.open('/etc/passwd').read()
```