## php7和php5的区别
参考：http://xiaoze.club/

### 函数方面
1. preg_replace()不再支持/e修饰符
但是有新的函数`preg_replace_callback("/.*/",function ($a){@eval($a[0]);},$_GET["h"]);`
2. create_function()被废弃
3. mysql_*系列全员移除
4. unserialize()增加一个可选白名单参数
类名也可以是布尔数据，如果是FALSE就会将所有的对象都转换为__PHP_Incomplete_Class对象。TRUE是无限制。也可以传入类名实现白名单
5. assert()默认不在可以执行代码

### 语法修改
1. foreach不再改变内部数组指针  在php7.0.0以后被改回去
2. 8进制字符容错率降低   在php7.0.0以后被改回去
在php5版本，如果一个八进制字符如果含有无效数字，该无效数字将被静默删节。
```php
<?php
echo octdec( '012999999999999' ) . "\n";
echo octdec( '012' ) . "\n";
if (octdec( '012999999999999' )==octdec( '012' )){
        echo ": )". "\n";
}
```
3. 十六进制字符串不再被认为是数字
目前截至最新的PHP7.3版本依然没有改回去的征兆，官方称不会在改了。
4. 超大浮点数类型转换截断
5. 可以使用{}代替[]

6. php7以上允许{}代替[]来取数组的元素
```php
<?php
echo $_GET{'asdf'};
?>
```

### 杂项
1. exec(), system() passthru()函数对 NULL 增加了保护。
2. list()不再能解开字符串string变量。
3. $HTTP_RAW_POST_DATA 被移除。
4. __autoload() 方法被废弃。
5. parse_str() 不加第二个参数会直接把字符串导入当前的符号表，如果加了就会转换成一个数组。现在是第二个参数是必选选项了。
6. 统一不同平台下的整型长度
7. session_start() 可以加入一个数组覆盖php.ini的配置
