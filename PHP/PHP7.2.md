# PHP7.2函数相关变更

## （废弃）__autoload() 方法

`__autoload()` 方法已被废弃， 因为和 `spl_autoload_register()` 相比功能较差 (因为无法链式处理多个 autoloader)， 而且也无法在两种 autoloading 样式中配合使用。

## （废弃）create_function() 函数

考虑到此函数的安全隐患问题（它是 [eval()](https://www.php.net/manual/zh/function.eval.php) 的瘦包装器），该过时的函数现在已被废弃。 更好的选择是[匿名函数](https://www.php.net/manual/zh/functions.anonymous.php)。

## （废弃）parse_str() 不加第二个参数

使用 `parse_str()` 时，不加第二个参数会导致查询字符串参数导入当前符号表。 考虑到安全隐患问题，不加第二个参数使用 `parse_str()` 的行为已被废弃。 此函数的第二个选项为必填项，它使查询字符串转为 Array。

## （废弃）assert() 一个字符串参数

`assert()` 字符串参数将要求它能被 `eval()` 执行。 考虑到可能被执行远程代码，废弃了字符串的 [assert()](https://www.php.net/manual/zh/function.assert.php)，最好提供 bool 的表达式。

## （移除） [utf8_encode()](https://www.php.net/manual/zh/function.utf8-encode.php) 和 [utf8_decode()](https://www.php.net/manual/zh/function.utf8-decode.php)[ ¶](https://www.php.net/manual/zh/migration72.other-changes.php#migration72.other-changes.utf8_-functions-to-ext-standard)

[utf8_encode()](https://www.php.net/manual/zh/function.utf8-encode.php) 和 [utf8_decode()](https://www.php.net/manual/zh/function.utf8-decode.php) 现在已经作为字符串处理函数移到标准扩展中， 在此之前需要启用 [XML](https://www.php.net/manual/zh/book.xml.php) 扩展才能使用。

# PHP7.2语法相关变更

## （增加）允许重写抽象方法

当一个抽象类继承于另外一个抽象类的时候，继承后的抽象类可以重写被继承的抽象类的抽象方法。

```php
<?php

abstract class A
{
    abstract function test(string $s);
}
abstract class B extends A
{
    // overridden - still maintaining contravariance for parameters and covariance for return
    abstract function test($s) : int;
}
```

## （增加）允许分组命名空间的尾部逗号

命名空间可以在PHP 7中使用尾随逗号进行分组引入。

```php
<?php

use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};
```

## （增加）转化数字键

转化数组为对象类型时，如果数组存在数字键，现在可以访问了。

```
<?php

// array to object
$arr = [0 => 1];
$obj = (object)$arr;
var_dump(
    $obj,
    $obj->{'0'}, // now accessible
    $obj->{0} // now accessible
);

# 结果
/*
PHP<7.2
object(stdClass)#1 (1) {
  [0]=>
  int(1)
}
NULL
NULL

PHP7.2
object(stdClass)#1 (1) {
  ["0"]=>
  int(1)
}
int(1)
int(1)
```

对象转为数组也同样可以了

```
<?php

// object to array
$obj = new class {
    public function __construct()
    {
        $this->{0} = 1;
    }
};
$arr = (array)$obj;
var_dump(
    $arr,
    $arr[0], // now accessible
    $arr['0'] // now accessible
);
```

## （废弃）不带引号的字符串

不带引号的字符串是不存在的全局常量，转化成他们自身的字符串。 在以前，该行为会产生 **E_NOTICE**，但现在会产生 **E_WARNING**。在下一个 PHP 主版本中，将抛出 **Error** 异常。

```php
<?php

var_dump(NONEXISTENT);

/* Output:
Warning: Use of undefined constant NONEXISTENT - assumed 'NONEXISTENT' (this will throw an Error in a future version of PHP) in %s on line %d
string(11) "NONEXISTENT"
*/
```

# 杂项

## 新的对象类型[ ¶](https://www.php.net/manual/zh/migration72.new-features.php#migration72.new-features.object-type)

## 通过名称加载扩展[ ¶](https://www.php.net/manual/zh/migration72.new-features.php#migration72.new-features.ext-loading-by-name)

## 扩展了参数类型[ ¶](https://www.php.net/manual/zh/migration72.new-features.php#migration72.new-features.param-type-widening)

## Disallow passing **NULL** to [get_class()](https://www.php.net/manual/zh/function.get-class.php)[ ¶](https://www.php.net/manual/zh/migration72.incompatible.php#migration72.incompatible.no-null-to-get_class)
