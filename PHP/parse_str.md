## parse_str

参考文章：
- https://secure.php.net/manual/zh/function.parse-str.php
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day7/files/README.md

函数结构：
```
parse_str ( string $encoded_string [, array &$result ] ) : void
```

使用缺陷：
可能存在变量覆盖。
