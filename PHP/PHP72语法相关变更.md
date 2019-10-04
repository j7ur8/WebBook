# PHP7.2语法相关变更

## 新特性

### 允许重写抽象方法

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

### 允许分组命名空间的尾部逗号

命名空间可以在PHP 7中使用尾随逗号进行分组引入。

```php
<?php

use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};
```

### 转化数字键

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

## 废弃的

### 不带引号的字符串

不带引号的字符串是不存在的全局常量，转化成他们自身的字符串。 在以前，该行为会产生 **E_NOTICE**，但现在会产生 **E_WARNING**。在下一个 PHP 主版本中，将抛出 **Error** 异常。

```php
<?php

var_dump(NONEXISTENT);

/* Output:
Warning: Use of undefined constant NONEXISTENT - assumed 'NONEXISTENT' (this will throw an Error in a future version of PHP) in %s on line %d
string(11) "NONEXISTENT"
*/
```

