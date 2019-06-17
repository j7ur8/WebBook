参考：
- https://wooyun.js.org/drops/user.ini%E6%96%87%E4%BB%B6%E6%9E%84%E6%88%90%E7%9A%84PHP%E5%90%8E%E9%97%A8.html
- https://php.net/manual/zh/configuration.file.per-user.php

利用范围：  
- 以fastcgi运行的php

## 简析
- user_ini.filename 设定了 PHP 会在每个目录下搜寻的文件名；如果设定为空字符串则 PHP 不会搜寻。默认值是 .user.ini。
- user_ini.cache_ttl 控制着重新读取用户 INI 文件的间隔时间。默认是 300 秒（5 分钟）。
- 除了PHP_INI_SYSTEM以外的模式（包括PHP_INI_ALL）都是可以通过.user.ini来设置的。

配置变量中有`auto_prepend_file`和`auto_apend_file`指定在文件前和文件尾包含文件。如:
```bash
auto_apend_file=01.gif
auto_prepend_file=01.gif
```
所以，我们可以借助.user.ini轻松让所有php文件都“自动”包含某个文件，而这个文件可以是一个正常php文件，也可以是一个包含一句话的webshell。