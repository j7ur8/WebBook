## php7和php5的区别
**参考**：

- 转载自http://xiaoze.club/

### 函数方面
#### preg_replace()不再支持/e修饰符

**测试代码**

```php
<?php
preg_replace("/.*/e",$_GET["h"],"."); 
?>
```

但是有新的函数`preg_replace_callback``

```php
<?php
$_GET["h"]=phpinfo();
preg_replace_callback("/.*/",function ($a){@eval($a[0]);},$_GET["h"]);
```

![](/images/19-7-22_PHP_PHP7和PHP5的区别_函数方面_1.png)

#### create_function()被废弃

**测试代码**

```php
<?php
$func =create_function('',$_POST['cmd']);$func();
?>
```

少了一种可以利用当后门的函数，实际上它是通过执行eval实现的。可有可无。  

#### mysql_*系列全员移除

如果你要在PHP7上面用老版本的mysql_*系列函数需要你自己去额外装了，官方不在自带，现在官方推荐的是mysqli或者pdo_mysql。

#### unserialize()增加一个可选白名单参数

可选白名单参数也可以是布尔数据，如果是FALSE就会将所有的对象都转换为__PHP_Incomplete_Class对象。TRUE是无限制。也可以传入类名实现白名单（ 还好现在是可选不是必选，要是默认FALSE逼程序员弄白名单那就真的吐血了。  ）

**函数构造**

```php
unserialize ( string $str [, array $options ] ) : mixed
```

**测试代码**

```php
<?php
class a{
	public $a='123';
}

$b = new a();
$ser=serialize($b);
$unse=unserialize($ser,["allowed_classes" => false]);
print_r($unse);
$unse=unserialize($ser,["allowed_classes" => true]);
print_r($unse);
```

![](/images/19-7-22_PHP_PHP7和PHP5的区别_函数方面_2.png)

#### session_start() 可以加入一个数组覆盖php.ini的配置

**函数构造**

```php
session_start ([ array $options = array() ] ) : bool
```

官方：

> | Version | Description                        |
> | ------- | ---------------------------------- |
> | 7.0.0   | The `options` parameter was added. |
>
> **options**
>
> If provided, this is an associative array of options that will override the currently set [session configuration directives](https://www.php.net/manual/en/session.configuration.php). The keys should not include the *session.* prefix.
>
> In addition to the normal set of configuration directives, a *read_and_close* option may also be provided. If set to **TRUE**, this will result in the session being closed immediately after being read, thereby avoiding unnecessary locking if the session data won't be changed.



#### 杂项

- exec(), system() passthru()函数对 NULL 增加了保护。

- list()不再能解开字符串string变量。

- $HTTP_RAW_POST_DATA 被移除。

- __autoload() 方法被废弃。

- parse_str() 不加第二个参数会直接把字符串导入当前的符号表，如果加了就会转换成一个数组。现在是第二个参数是必选选项了。

  





### 语法修改
#### foreach不再改变内部数组指针  在php7.0.0以后被改回去

这个问题在PHP7.0.0以后的版本又被改回去了，只影响这一个版本。  

**测试代码**

```php
<?php
$a = array('1','2','3');
foreach ($a as $k=>&$n){
    echo "";
}
print_r($a);
foreach ($a as $k=>$n){
    echo "";
}
print_r($a);

//php5和php7.0.0的结果
/*
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
)
Array
(
    [0] => 1
    [1] => 2
    [2] => 2
)
*/
```



#### 8进制字符容错率降低   

在php5版本，如果一个八进制字符如果含有无效数字，该无效数字将被静默删节。但是在php7.0.0里面会触发一个解析错误。同样在php7.0.0以后被改回去。

**测试代码**

```php
<?php
// 在php5和php7.0.0之外的版本测试结果应该如下
echo octdec( '012999999999999' ) . "\n";    // 10
echo octdec( '012' ) . "\n";                // 10
if (octdec( '012999999999999' )==octdec( '012' )){
        echo ": )". "\n";                   // : )
}
```
#### 十六进制字符串不再被认为是数字

目前截至最新的PHP7.3版本依然没有改回去的征兆，官方称不会在改了。

**测试代码**

```php
<?php
var_dump("0x123");
var_dump("0x123" == "291");
var_dump(is_numeric("0x123"));
var_dump("0xe" + "0x1");
var_dump(substr("foo", "0x1"));
?>
```

![](/images/19-7-22_PHP_PHP7和PHP5的区别_语法修改_1.png)

左边php5.6，右边php7.2

### 移除了 ASP 和 script PHP 标签

![](/images/19-7-22_PHP_PHP7和PHP5的区别_语法修改_2.png)

#### 超大浮点数类型转换截断

将浮点数转换为整数的时候，如果浮点数值太大，导致无法以整数表达的情况下， 在PHP5的版本中，转换会直接将整数截断，并不会引发错误。 在PHP7中，会报错。  

#### php7以上允许{}代替[]来取数组的元素

```php
<?php
echo $_GET{'asdf'};
?>
```

#### 杂项

- 统一不同平台下的整型长度