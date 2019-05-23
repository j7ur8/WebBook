## in_array

函数结构：
in_array — 检查数组中是否存在某个值
```
in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] ) : bool
```

使用缺陷：

第三个参数`$strict`如果没设置成**TRUE**，就不会进行强检查。

利用条件：
- `$strict=FALSE`会造成`7=7shelll.php`