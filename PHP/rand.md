## rand

参考文章：
- https://xz.aliyun.com/t/3656#toc-3
- https://secure.php.net/manual/zh/function.rand.php

函数结构：
```
rand ( void ) : int ;  rand ( int $min , int $max ) : int
```

使用缺陷：
拿到种子或者随机数可以进行爆破
工具：http://www.openwall.com/php_mt_seed/
