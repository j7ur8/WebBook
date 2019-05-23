## 文件包含
(PHP 5 >= 5.3.0, PHP 7, PECL phar >= 1.0.0)

参考：
	- https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/
利用条件
	- phar文件要能够上传到服务器端。
	- 存在魔术方法作为跳板
	- 存在相关[函数](https://blog.zsxsoft.com/post/38)函数,并且参数可控，:、/、phar等特殊字符没有被过滤。
	  ![](/images/19-1-19_2018总结-PHP篇_利用类进行反序列化Phar1.png)
index.php
```php
<?php
include($_GET['file']);
?>
```
1.txt
```txt
<?php phpinfo() ?>
```
`http://127.0.0.1/exp.zip/1.txt`

## 反序列化
参考
	- https://blog.zsxsoft.com/post/38
	- https://paper.seebug.org/680/