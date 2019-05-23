## filter_var
filter_var — 使用特定的过滤器过滤一个变量

参考文章：
- https://secure.php.net/manual/zh/function.filter-var.php
- https://www.anquanke.com/post/id/101058
- https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html#0x03-filter_validate_email
- https://stackoverflow.com/questions/19220158/php-filter-validate-email-does-not-work-correctly

函数结构：
```
filter_var ( mixed $variable [, int $filter = FILTER_DEFAULT [, mixed $options ]] ) : mixed
```
使用缺陷：
1. `FILTER_VALIDATE_URL`

`filter_var('javascript://comment%250Aalert(1)', FILTER_VALIDATE_URL);`会导致XSS攻击。因为//在JavaScript中表示单行注释，%250a解码后为换行符，所以alert(1)和//不在一行，就可以执行。
在如下脚本中：
	```
	<?php
	   echo "Argument: ".$argv[1]."n";
	   // check if argument is a valid URL
	   if(filter_var($argv[1], FILTER_VALIDATE_URL)) {
	      // parse URL
	      $r = parse_url($argv[1]);
	      print_r($r);
	      // check if host ends with google.com
	      if(preg_match('/google.com$/', $r['host'])) {
	         // get page from URL
	         exec('curl -v -s "'.$r['host'].'"', $a);
	         print_r($a);
	      } else {
	         echo "Error: Host not allowed";
	      }
	   } else {
	      echo "Error: Invalid URL";
	   }
	?>
	```
可以利用`curl`和`filter_var`之间的解析差异绕过：
`0://evil.com:80;google.com:80/`
`0://evil.com:80,google.com:80/`

2. `FILTER_VALIDATE_EMAIL`

绕过方法为：`"Joe'Blow"@example.com`