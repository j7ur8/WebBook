## extract

**参考文章：**  
- https://secure.php.net/manual/zh/function.extract.php
- https://crayon-xin.github.io/2018/05/21/extract%E5%8F%98%E9%87%8F%E8%A6%86%E7%9B%96/
- https://www.freebuf.com/column/150731.html

**函数结构：**  
```
extract ( array &$array [, int $flags = EXTR_OVERWRITE [, string $prefix = NULL ]] ) : int
```

**使用缺陷：**   
因为`$flags`的默认参数是`EXTR_OVERWRITE`,所以可能导致变量覆盖漏洞

### 解析
简单如下
```php
<?php
$j7ur8='best';
$arr['j7ur8']='the best';
extract($arr);
print_r($j7ur8);  # the best
```

extract将数组$arr中对应的`key=>value`注册成`$key=value`。而由于extract函数的`$flags`默认值为`EXTR_OVERWRITE`，所以会覆盖已存在的的相同变量的值。