# PHP7.1函数相关变更

## 变化的函数

### list()现在支持键名

现在`list()`和它的新的*[]*语法支持在它内部去指定键名。这意味着它可以将任意类型的数组 都赋值给一些变量（与短数组语法类似）

```php
<?php
$data = [
    ["id" => 1, "name" => 'Tom'],
    ["id" => 2, "name" => 'Fred'],
];

// list() style
list("id" => $id1, "name" => $name1) = $data[0];

// [] style
["id" => $id1, "name" => $name1] = $data[0];

// list() style
foreach ($data as list("id" => $id, "name" => $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as ["id" => $id, "name" => $name]) {
    // logic here with $id and $name
}
```

### 禁止动态调用的函数

如下：

- [assert()](https://www.php.net/manual/zh/function.assert.php) - with a string as the first argument
- [compact()](https://www.php.net/manual/zh/function.compact.php)
- [extract()](https://www.php.net/manual/zh/function.extract.php)
- [func_get_args()](https://www.php.net/manual/zh/function.func-get-args.php)
- [func_get_arg()](https://www.php.net/manual/zh/function.func-get-arg.php)
- [func_num_args()](https://www.php.net/manual/zh/function.func-num-args.php)
- [get_defined_vars()](https://www.php.net/manual/zh/function.get-defined-vars.php)
- [mb_parse_str()](https://www.php.net/manual/zh/function.mb-parse-str.php) - with one arg
- [parse_str()](https://www.php.net/manual/zh/function.parse-str.php) - with one arg

尾随的`-`代表禁止动态调用的条件。`func_get_args`等几个函数测试了下好像并没有禁止动态调用？不太懂。

### unserialize

对`allowed_classes`参数严格化，如果对其传入的值是**布尔值**和**数组**之外的内容，`unserialize`将返回**False**和**E_WARNING**

### getenv可以不需要传入参数

`getenv()` 可以不传入任何参数。 如果不传入参数，此函数会以关联数组的形式 返回所有的环境变量。

![](../images/19-10-1_PHP_PHP71函数相关变更_2.png)

### parse_url支持RFC3986

`parse_url()` 更加严格的限制， 并且提供对 RFC3986 的支持。

### session_start将返回false

`session_start()` 当无法成功初始化会话的时候，返回 **FALSE**， 并且不会初始化超级变量 `$_SESSION`。

但是我测试的时候发现，PHP7.0已经会返回False了（Win+PHP7.12nts

```php
<?php
@var_dump(session_start('11'));
@var_dump($_SESSION);

# 结果
/*
php5
bool(true)
array(0) {
}

PHP5
bool(false)
NULL
```



## 废弃

### 两个函数的Eval选项

[mb_ereg_replace()](https://www.php.net/manual/zh/function.mb-ereg-replace.php)和[mb_eregi_replace()](https://www.php.net/manual/zh/function.mb-eregi-replace.php)

### ext/mcrypt