# in_array

**函数结构：**

```
in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] ) : bool
```

## 弱比较

第三个参数`$strict`如果没设置成**TRUE**，就不会进行强检查。

**利用条件：**

- `$strict=FALSE`

**测试代码：**

```php
<?php
$a=7;
$b[]='7shell';
var_dump(in_array($a,$b));
var_dump(in_array($a,$b,$strict=true));
```

![](/images/19-7-23_PHP_in-array_1.png)

