# PHP中的变量

## 参考

- https://php.net/manual/en/language.variables.variable.php
- https://stackoverflow.com/questions/16408037/php-this-var-what-does-that-mean/16408056

## 可变变量

### 可变属性名

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
echo $foo->{$start . $end} . "\n";

$arr = 'arr';
echo $foo->{$arr[1]} . "\n";

/*
I am bar.
I am bar.
I am bar.
I am r.
*/
?>
```

## 自增自减符号

```php
<?php
$a='';
echo --$a;
echo ++$a;

# 结果
/*
-1
1
*/
```

## {}代替[]来取数组的元素

```php
<?php
echo $_GET{'asdf'};
?>
```

### 