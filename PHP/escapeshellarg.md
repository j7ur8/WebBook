# escapeshellarg和escapeshellcmd

**参考文章：**

- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day5/files/README.md
- https://secure.php.net/manual/zh/function.escapeshellarg.php
- https://secure.php.net/manual/zh/function.escapeshellcmd.php

**函数结构：**
escapeshellarg — 把字符串转码为可以在 shell 命令里使用的参数
escapeshellcmd — shell 元字符转义
```
escapeshellarg ( string $arg ) : string
escapeshellcmd ( string $command ) : string
```

## 利用

mail()函数底层实现了`escapeshellcmd`函数，对用户输入的邮箱进行检测。
但是如果escapeshellcmd和escapeshellarg一起使用，就会造成特殊字符逃逸。
**测试代码：**

```php
<?php
$param="127.0.0.1' -v -d a=1";
$a=escapeshellarg($param);
$b=escapeshellcmd($a);
$cmd="curl ".$b;
var_dump($a)."\n";
var_dump($a)."\n";
var_dump($cmd)."\n";
?>
```

分析如下：
![](/images/19-1-20_2018总结-PHP篇_escapeshellcmd1.jpg)
![](/images/19-7-9_PHP_escapeshellcmd2.png)

但是windows测试如下，有点问题
![](/images/19-7-9_PHP_escapeshellcmd3.jpg)