## preg_replace

漏洞存在于PHP<7的版本

参考文章：
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day8/files/README.md
- https://xz.aliyun.com/t/2557

函数结构：
```
preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] ) : mixed
```

使用缺陷：
$pattern 存在 /e 模式修正符，允许代码执行