## extract

参考文章：
- https://secure.php.net/manual/zh/function.extract.php
- https://crayon-xin.github.io/2018/05/21/extract%E5%8F%98%E9%87%8F%E8%A6%86%E7%9B%96/
- https://www.freebuf.com/column/150731.html

函数结构：
```
extract ( array &$array [, int $flags = EXTR_OVERWRITE [, string $prefix = NULL ]] ) : int
```

使用缺陷：
可能导致变量覆盖漏洞