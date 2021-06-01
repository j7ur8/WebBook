## import_request_variables变量覆盖

**范围：**
- (PHP 4 >= 4.1.0, PHP 5 < 5.4.0)

**参考：**
- https://www.php.net/manual/zh/function.import-request-variables.php
- https://p0sec.net/index.php/archives/35/

### 解析

`import_request_variables`涉及到`php.ini`中`register_global`的配置。`register_global`在`php.ini`中默认配置为0。  
  
我们可以看以下代码：
```php
<?php  
echo "Register_globals: ".(int)ini_get("register_globals")."<br/>";  
  
if ($auth){  
   echo "private!";  
}  
?>
```
当我们把register_globals设置为1时，访问`http://127.0.0.1?auth=1`，将会自动注册变量`$auth`。  
但如果代码中已经对`$auth`赋了初始值`$auth=0`，那么将不会变量覆盖。
  
但默认情况下`register_global`为0，所以就有了`import_request_variables`函数。
```php
<?php  
$auth = '0';  
import_request_variables('G');  
  
if($auth == 1){  
  echo "private!";  
}else{  
  echo "public!";  
}  
?>  
```
