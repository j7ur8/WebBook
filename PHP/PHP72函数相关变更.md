# 函数

## 废弃的

### __autoload() 方法

`__autoload()` 方法已被废弃， 因为和 `spl_autoload_register()` 相比功能较差 (因为无法链式处理多个 autoloader)， 而且也无法在两种 autoloading 样式中配合使用。

### create_function() 函数

考虑到此函数的安全隐患问题（它是 [eval()](https://www.php.net/manual/zh/function.eval.php) 的瘦包装器），该过时的函数现在已被废弃。 更好的选择是[匿名函数](https://www.php.net/manual/zh/functions.anonymous.php)。

### parse_str() 不加第二个参数

使用 `parse_str()` 时，不加第二个参数会导致查询字符串参数导入当前符号表。 考虑到安全隐患问题，不加第二个参数使用 `parse_str()` 的行为已被废弃。 此函数的第二个选项为必填项，它使查询字符串转为 Array。

### assert() 一个字符串参数

`assert()` 字符串参数将要求它能被 `eval()` 执行。 考虑到可能被执行远程代码，废弃了字符串的 [assert()](https://www.php.net/manual/zh/function.assert.php)，最好提供 bool 的表达式。

## 移除的

### 移动 [utf8_encode()](https://www.php.net/manual/zh/function.utf8-encode.php) 和 [utf8_decode()](https://www.php.net/manual/zh/function.utf8-decode.php)[ ¶](https://www.php.net/manual/zh/migration72.other-changes.php#migration72.other-changes.utf8_-functions-to-ext-standard)

[utf8_encode()](https://www.php.net/manual/zh/function.utf8-encode.php) 和 [utf8_decode()](https://www.php.net/manual/zh/function.utf8-decode.php) 现在已经作为字符串处理函数移到标准扩展中， 在此之前需要启用 [XML](https://www.php.net/manual/zh/book.xml.php) 扩展才能使用。