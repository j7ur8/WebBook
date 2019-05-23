## htmlentities

参考文章：
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day12/files/README.md
- http://php.net/manual/zh/function.htmlentities.php

函数结构：
```
string htmlentities ( string $string [, int $flags = ENT_COMPAT | ENT_HTML401 [, string $encoding = ini_get("default_charset") [, bool $double_encode = true ]]] )
```

使用缺陷：
默认情况下只会过滤双引号
