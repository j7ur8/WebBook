## strpos
参考文章：
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day4/files/README.md

函数结构：
```
strpos ( string $haystack , mixed $needle [, int $offset = 0 ] ) : int
```

使用缺陷：

如果查询到的字符在第一位，那么返回为数字为0；而查询不到的结果也为false。
0和false取反皆为true，如果不注意就有可能造成错误。