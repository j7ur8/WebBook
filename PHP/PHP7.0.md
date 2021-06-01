# 函数变更

参考：

- https://www.php.net/manual/zh/appendices.php
- http://xiaoze.club/

## 关于list处理方式的变更

**list不再以反向的顺序来进行赋值**

`list`现在会按照变量定义的顺序来给他们进行赋值，而非反过来的顺序。 通常来说，这只会影响`list`与数组的`[]`操作符一起使用的案例，如下所示：

```php
<?php
list($a[], $a[], $a[]) = [1, 2, 3];
var_dump($a);
?>
```

```php
PHP5
array(3) {
  [0]=>
  int(3)
  [1]=>
  int(2)
  [2]=>
  int(1)
}

PHP7
array(3) {
  [0]=>
  int(1)
  [1]=>
  int(2)
  [2]=>
  int(3)
}
```

推荐不要依赖l`ist`的赋值顺序，因为这是一个在未来也许会变更的实现细节。

**list 结构现在不再能是空的**

list结构现在不再能是空的。如下的例子不再被允许：

```php
<?php
var_dump(list() = $a);
var_dump(list(,,) = $a);
var_dump(list($x, list(), $y) = $a);
var_dump($x)
?>
```

**list不再能解开string**

`list`不再能解开字符串（string）变量。 你可以使用`str_split`来代替它

## foreach的变化

`foreach`发生了细微的变化，控制结构， 主要围绕阵列的内部数组指针和迭代处理的修改。

**foreach不再改变内部数组指针**

在PHP7之前，当数组通过`foreach`迭代时，数组指针会移动。现在开始，不再如此，见下面代码

```php
<?php
$array = [0, 1, 2];
foreach ($array as &$val) {
    var_dump(current($array));
}
?>
```

结果

```php
PHP5
int(1)
int(2)
bool(false)
    
PHP7
int(0)
int(0)
int(0)
```

**foreach通过值遍历时，操作的值为数组的副本**

当默认使用通过值遍历数组时，`foreach`实际操作的是数组的迭代副本，而非数组本身。这就意味着，`foreach` 中的操作不会修改原数组的值。

**foreach通过引用遍历时，有更好的迭代特性**

当使用引用遍历数组时，现在 `foreach`在迭代中能更好的跟踪变化。例如，在迭代中添加一个迭代值到数组中，参考下面的代码：

```php
<?php
$array = [0];
foreach ($array as &$val) {
    var_dump($val);
    $array[1] = 1;
}
?>
```

结果

```
PHP5
int(0)

PHP7
int(0)
int(1)
```

## filter_var转换十六进制数字

`filter_var` 函数可以用于检查一个 `string` 是否含有十六进制数字,并将其转换为`integer`:

```php
<?php
$str = "0xffff";
$int = filter_var($str, FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX);
if (false === $int) {
    throw new Exception("Invalid integer!");
}
var_dump($int); // int(65535)
?>
```

## 通过 define 定义常量数组

Array 类型的常量现在可以通过 `define()` 来定义。在 PHP5.6 中仅能通过 `const` 定义。

```php
<?phpdefine('ANIMALS', [    'dog',    'cat',    'bird']);echo ANIMALS[1]; // 输出 "cat"?>
```

## 为unserialize()提供过滤

这个特性旨在提供更安全的方式解包不可靠的数据。它通过白名单的方式来防止潜在的代码注入。

可选白名单参数也可以是布尔数据，如果是FALSE就会将所有的对象都转换为`__PHP_Incomplete_Class`对象。TRUE是无限制。也可以传入类名实现白名单。

```php
<?phpclass a{	public $a='123';}$b = new a();$ser=serialize($b);$unse=unserialize($ser,["allowed_classes" => false]);print_r($unse);$unse=unserialize($ser,["allowed_classes" => true]);print_r($unse);
```

![](/Users/j7ur8/WebBook/BookBuild/images/19-7-22_PHP_PHP7和PHP5的区别_函数方面_2.png)

## session_start可以加入一个数组

`session_start()` 可以接受一个 `array` 作为参数， 用来覆盖 php.ini 文件中设置的 [会话配置选项](https://www.php.net/manual/zh/session.configuration.php)。

在调用 `session_start()` 的时候， 传入的选项参数中也支持 `session.lazy_write` 行为， 默认情况下这个配置项是打开的。它的作用是控制 PHP 只有在会话中的数据发生变化的时候才 写入会话存储文件，如果会话中的数据没有发生改变，那么 PHP 会在读取完会话数据之后， 立即关闭会话存储文件，不做任何修改，可以通过设置 `read_and_close`来实现。

例如，下列代码设置 `session.cache_limiter` 为 *private*，并且在读取完毕会话数据之后马上关闭会话存储文件。

```php
<?phpsession_start([    'cache_limiter' => 'private',    'read_and_close' => true,]);?>
```

## preg_replaces不再支持/e修饰符

```php
<?php$_GET["h"]=phpinfo();preg_replace_callback("/.*/",function ($a){@eval($a[0]);},$_GET["h"]);
```

![](/Users/j7ur8/WebBook/BookBuild/images/19-7-22_PHP_PHP7和PHP5的区别_函数方面_1.png)

但是有新的函数`preg_replace_callback`

## dirname增加了参数

`dirname()` 增加了可选的第二个参数, `depth`, 获取当前目录向上 `depth` 级父目录的名称

```php
<?php$a=dirname('c:/a/b/c/d',1);print_r($a);#结果/*c:/a/b/c*/
```

## system等函数对NULL增加了保护.

`exec()`, `system()` and `passthru()` 函数对 NULL 增加了保护（我也不知道啥意思。

## 增加整数除法函数 intdiv

新加的函数 `intdiv()` 用来进行 整数的除法运算。

```php
<?phpvar_dump(intdiv(10, 3));// int(3)?>
```

## 增加preg_replace_callback_array

在 PHP 7 之前，当使用 `preg_replace_callback()` 函数的时候， 由于针对每个正则表达式都要执行回调函数，可能导致过多的分支代码。 而使用新加的 `preg_replace_callback_array()` 函数， 可以使得代码更加简洁。

现在，可以使用一个关联数组来对每个正则表达式注册回调函数， 正则表达式本身作为关联数组的键， 而对应的回调函数就是关联数组的值。

```php
<?php$subject = 'Aaaaaa Bbb';preg_replace_callback_array(    [        '~[a]+~i' => function ($match) {            echo strlen($match[0]), ' matches for "a" found', PHP_EOL;        },        '~[b]+~i' => function ($match) {            echo strlen($match[0]), ' matches for "b" found', PHP_EOL;        }    ],    $subject);    /*6 matches for "a" found3 matches for "b" found*/
```

## 增加Closure::call()

**Closure::call()** 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。

```php
<?phpclass A {private $x = 1;}// PHP 7 之前版本的代码$getXCB = function() {return $this->x;};$getX = $getXCB->bindTo(new A, 'A'); // 中间层闭包echo $getX();// PHP 7+ 及更高版本的代码$getX = function() {return $this->x;};echo $getX->call(new A);#结果/*11*/
```

## 被移除的函数

- `call_user_method`和`call_user_method_array`

- 所有的`ereg*`函数

- `mcrypt`的一部分废弃函数

- 所有`ext/mysql`函数

- 所有`ext/mssql`函数

- dl in PHP-FPM

  dl()在 PHP-FPM 不再可用，在 CLI 和 embed SAPIs 中仍可用。

# 语法变更

## 关于变量处理的变化

PHP 7 现在使用了抽象语法树来解析源代码。这使得许多由于之前的PHP的解释器的限制所不可能实现的改进得以实现。 但出于一致性的原因导致了一些特殊例子的变动，而这些变动打破了向后兼容。 在这一章中将详细介绍这些例子。

## 关于间接使用变量、属性和方法的变化

对变量、属性和方法的间接调用现在将严格遵循从左到右的顺序来解析，而不是之前的混杂着几个特殊案例的情况。 下面这张表说明了这个解析顺序的变化。

​										**间接调用的表达式的新旧解析顺序**

| 表达式                | PHP 5 的解析方式        | PHP 7 的解析方式        |
| :-------------------- | :---------------------- | :---------------------- |
| `$$foo['bar']['baz']` | `${$foo['bar']['baz']}` | `($$foo)['bar']['baz']` |
| `$foo->$bar['baz']`   | `$foo->{$bar['baz']}`   | `($foo->$bar)['baz']`   |
| `$foo->$bar['baz']()` | `$foo->{$bar['baz']}()` | `($foo->$bar)['baz']()` |
| `Foo::$bar['baz']()`  | `Foo::{$bar['baz']}()`  | `(Foo::$bar)['baz']()`  |

使用了旧的从右到左的解析顺序的代码必须被重写，明确的使用大括号来表明顺序（参见上表中间一列）。 这样使得代码既保持了与PHP 7.x的前向兼容性，又保持了与PHP 5.x的后向兼容性。

这同样影响了[**global**](https://www.php.net/manual/zh/language.variables.scope.php#language.variables.scope.global) 关键字。如果需要的话可以使用大括号来模拟之前的行为。

```php
<?php
function f() {
    // Valid in PHP 5 only.
    global $$foo->bar;

    // Valid in PHP 5 and 7.
    global ${$foo->bar};
}
?>
```

## 8进制字符容错率降低   

在php5版本，如果一个八进制字符如果含有无效数字，该无效数字将被静默删节。但是在php7.0.0里面会触发一个解析错误。同样在php7.0.0以后被改回去。

```php
<?php
// 在php5和php7.0.0之外的版本测试结果应该如下
echo octdec( '012999999999999' ) . "\n";    // 10
echo octdec( '012' ) . "\n";                // 10
if (octdec( '012999999999999' )==octdec( '012' )){
        echo ": )". "\n";                   // : )
}
```

## 负位移运算

以负数形式进行的位移运算将会抛出一个 **ArithmeticError**：

```php
<?php
var_dump(1 >> -1);
?>
```

结果

```
PHP5
int(0)

PHP7
Fatal error: Uncaught ArithmeticError: Bit shift by negative number in /tmp/test.php:2
Stack trace:
#0 {main}
  thrown in /tmp/test.php on line 2

```

## 超范围后产生位移

超出 `integer`位宽的位移操作（无论哪个方向）将始终得到 0 作为结果。在从前，这一操作是结构依赖的

## 十六进制字符串规范

十六进制字符串不再被认为是数字

目前截至最新的PHP7.3版本依然没有改回去的征兆，官方称不会在改了。

```php
<?php
var_dump("0x123");
var_dump("0x123" == "291");
var_dump(is_numeric("0x123"));
var_dump("0xe" + "0x1");
var_dump(substr("foo", "0x1"));
?>
```

结果:

左边php5.6，右边php7.2

![](/Users/j7ur8/WebBook/BookBuild/images/19-7-22_PHP_PHP7和PHP5的区别_语法修改_1.png)



[filter_var()](https://www.php.net/manual/zh/function.filter-var.php) 函数可以用于检查一个 [string](https://www.php.net/manual/zh/language.types.string.php) 是否含有十六进制数字,并将其转换为[integer](https://www.php.net/manual/zh/language.types.integer.php):

```php
<?php
$str = "0xffff";
$int = filter_var($str, FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX);
if (false === $int) {
    throw new Exception("Invalid integer!");
}
var_dump($int); // int(65535)
?>
```

由于新的 [Unicode codepoint escape syntax](https://www.php.net/manual/zh/migration70.new-features.php#migration70.new-features.unicode-codepoint-escape-syntax)语法， 紧连着无效序列并包含*\u{* 的字串可能引起致命错误。 为了避免这一报错，应该避免反斜杠开头。

## 移除了 ASP 和 script PHP 标签

使用类似 ASP 的标签，以及 script 标签来区分 PHP 代码的方式被移除。 受到影响的标签有：

| 开标签                    | 闭标签      |
| :------------------------ | :---------- |
| `<%`                      | `%>`        |
| `<%=`                     | `%>`        |
| `<script language="php">` | `</script>` |

## yield 变更为右联接运算符

在使用 `yield` 关键字的时候，不再需要括号， 并且它变更为右联接操作符，其运算符优先级介于 *print* 和 *=>* 之间。 这可能导致现有代码的行为发生改变：

```php
<?php
echo yield -1;
// 在之前版本中会被解释为：
echo (yield) - 1;
// 现在，它将被解释为：
echo yield (-1);

yield $foo or die;
// 在之前版本中会被解释为：
yield ($foo or die);
// 现在，它将被解释为：
(yield $foo) or die;
?>
```

可以通过使用括号来消除歧义。

## 关于对引用数组元素创建的变量的顺序变化

有如下代码

```php
<?php
$array = [];
$array["a"] =& $array["b"];
$array["b"] = 1;
var_dump($array);
?>
```

结果

```
PHP5
array(2) {
  ["b"]=>
  &int(1)
  ["a"]=>
  &int(1)
}

PHP7
array(2) {
  ["a"]=>
  &int(1)
  ["b"]=>
  &int(1)
}
```

## NULL合并运算符

由于日常使用中存在大量同时使用三元表达式和 `isset()`的情况， 我们添加了null合并运算符 (*??*) 这个语法糖。如果变量存在且值不为**NULL**， 它就会返回自身的值，否则返回它的第二个操作数。

```php
<?php
$sss=NULL;
$username=$sss ?? 'aaa';
echo $username."\n";

$sss= '';
$username=$sss ?? 'aaa';
echo $username."\n";

$username=$bbb ?? 'aaa';
echo $username."\n";

#结果
/*
aaa

aaa
*/
```

## 太空船操作符（组合比较符）

太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的[**常规比较规则**](https://www.php.net/manual/zh/types.comparisons.php)进行的。

```php
<?php
// 整数
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```

## 匿名类

现在支持通过*new class* 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。

```php
<?php
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});

var_dump($app->getLogger());

#结果
/*
object(class@anonymous)#2 (0) {
}
*/
```

## Unicode codepoint 转译语法

这接受一个以16进制形式的 Unicode codepoint，并打印出一个双引号或heredoc包围的 UTF-8 编码格式的字符串。 可以接受任何有效的 codepoint，并且开头的 0 是可以省略的。

```php
<?php
echo "\u{aa}";
echo "\u{0000aa}";
echo "\u{9999}";
```

## 导入use声名组

从同一 `namespace` 导入的类、函数和常量现在可以通过单个 [*use*](https://www.php.net/manual/zh/language.namespaces.importing.php) 语句 一次性导入了。

```php
<?php

// PHP 7 之前的代码
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ 及更高版本的代码
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```

## 放宽了保留词限制

现在允许全局保留词用于类/接口/Trait 中的属性、常量和方法名。 在引入新关键词时，此变更减少了对向后兼容的破坏，避免了 API 命名的限制。

使用流畅的接口实现内部 DSL 时，这非常有用：

```php
<?php
// 以前不能用  'new'、'private' 和 'for'
Project::new('Project Name')->private()->for('purpose here')->with('username here');
?>
```

唯一的限制是： *class*关键词不能用于常量名，否则会和 类名解析语法冲突 (*ClassName::class*)。

## （弃用）PHP4 风格的构造函数

PHP4 风格的构造函数（方法名和类名一样）将被弃用，并在将来移除。 如果在类中仅使用了 PHP4 风格的构造函数，PHP7 会产生 **E_DEPRECATED** 警告。 如果还定义了 **__construct()** 方法则不受影响。

```php
<?php
class foo {
    function foo() {
        echo 'I am the constructor';
    }
}
?>
```

## （弃用）静态调用非静态的方法

废弃了**静态**调用未声明成 **static** 的方法，未来可能会彻底移除该功能。

```php
<?php
class foo {
    function bar() {
        echo 'I am not static!';
    }
}

foo::bar();
?>
```

# 杂项

## 错误和异常处理相关的变更[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.error-handling)

## $HTTP_RAW_POST_DATA 被移除[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.http-raw-post-data)

## 从不匹配的上下文发起调用[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.incompatible-this)

## 函数定义不可以包含多个同名参数[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.func-parameters)

## 在函数中检视参数值会返回 *当前* 的值[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.func-parameter-modified)

## INI 文件中 *#* 注释格式被移除[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.ini-comments)

## Switch 语句不可以包含多个 default 块[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.multiple-default)

## JSON 扩展已经被 JSOND 取代[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.json-to-jsond)

## 在数值溢出的时候，内部函数将会失败[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.internal-function-failure-overflow)

## 自定义会话处理器的返回值修复[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.fixes-custom-session-handler)

## 相等的元素在排序时的顺序问题[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.sort-order)

## 错误的使用 break 和 switch 语句[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.break-continue)

## Mhash 不再是一个单独的扩展了[ ¶](https://www.php.net/manual/zh/migration70.incompatible.php#migration70.incompatible.other.mhash)

## 标量类型声明[ ¶](https://www.php.net/manual/zh/migration70.new-features.php#migration70.new-features.scalar-type-declarations)

## 返回值类型声明[ ¶](https://www.php.net/manual/zh/migration70.new-features.php#migration70.new-features.return-type-declarations)

## 生成器可以返回表达式[ ¶](https://www.php.net/manual/zh/migration70.new-features.php#migration70.new-features.generator-return-expressions)

## 允许在克隆表达式上访问对象成员[ ¶](https://www.php.net/manual/zh/migration70.new-features.php#migration70.new-features.others)