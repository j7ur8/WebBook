参考文章：
- http://www.laruence.com/2010/06/08/1579.html
- https://www.leavesongs.com/PENETRATION/use-pcre-backtrack-limit-to-bypass-restrict.html

条件：
- 非贪婪模式

demo
```php
<?php
function is_php($data){  
    return preg_match('/<\?.*[(`;?>].*/is', $data);  
}

if(!is_php($input)) {
    // fwrite($f, $input); ...
}
```

exp
```php
var_dump(preg_match('/union.+?select/is','union /*'.str_repeat('a', 10000000).'*/select'));
```

匹配下列字符串的回溯过程如下：
![](19-6-12_PHP_pcre回溯问题.png)

php限制了回溯的上限，其上限由php.ini的pcre.backtrack_limit定义。可以通过`var_dump(ini_get('pcre.backtrack_limit'))`查看当前环境的上限值。一旦正则匹配的回溯次数超过改值就会返回`bool(false)`。
  
如果用preg_match对字符串进行匹配，没有使用===全等号来判断返回值。则可能出现绕过。