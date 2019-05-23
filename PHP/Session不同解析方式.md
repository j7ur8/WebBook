## Session的不同解析方式
session.serialize_handler可以设置序列化/解序列化的处理器名字
如果序列化和反序列化的处理器不同，那么可能造成漏洞。

如LCTF2018_php's revenge

参考文章：
- https://secure.php.net/manual/zh/session.configuration.php#ini.session.serialize-handler
- http://hu3sky.ooo/2018/05/09/php%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%AD%A6%E4%B9%A0/
