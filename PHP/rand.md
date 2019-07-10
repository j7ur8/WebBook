## rand

**参考文章：**
- https://xz.aliyun.com/t/3656#toc-3
- https://secure.php.net/manual/zh/function.rand.php

**函数结构：**
```
rand ( void ) : int ;  rand ( int $min , int $max ) : int
```

**使用缺陷：**
拿到种子或者随机数可以进行爆破

**工具：**
- http://www.openwall.com/php_mt_seed/

### 工具使用
参考：
- https://www.openwall.com/php_mt_seed/README

一般情况，我们使用`mt_srand(534142874 ); echo mt_rand(), "\n";`生成的伪随机数，可以简单的使用
```bash
php_mt_seed 1328851649
```
得到结果：
![](/images/19-7-9_PHP_rand_tool-for-use_1.png)
可以发现得到了很多结果，参见README手册我们可以得知
![](/images/19-7-9_PHP_rand_tool-for-use_2.png)

改善命令
```bash
time php_mt_seed 1328851649 1328851649 0 2147483647  1423851145
```
![](/images/19-7-9_PHP_rand_tool-for-use_3.png)
虽然速度没有太多提升，但是派出了多余的值。

在上面同时介绍了高级用法，当出现了4个以上的参数时，`php_mt_seed`将把参数分成4个一组。每组按照处理4个的方法进行处理（见图片）。

**例子：**  
```php
<?php
//生成优惠码
$_SESSION['seed']=rand(0,999999999);
function youhuima(){
    mt_srand($_SESSION['seed']);
    $str_rand = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    $auth='';
    $len=15;
    for ( $i = 0; $i < $len; $i++ ){
        if($i<=($len/2))
              $auth.=substr($str_rand,mt_rand(0, strlen($str_rand) - 1), 1);
        else
              $auth.=substr($str_rand,(mt_rand(0, strlen($str_rand) - 1))*-1, 1);
    }
    setcookie('Auth', $auth);
}
//support
    if (preg_match("/^\d+\.\d+\.\d+\.\d+$/im",$ip)){
        if (!preg_match("/\?|flag|}|cat|echo|\*/i",$ip)){
               //执行命令
        }else {
              //flag字段和某些字符被过滤!
        }
    }else{
             // 你的输入不正确!
    }
?>
```

此代码根据生成的(0,61)之间伪随机数，截取`$str_rand`字符凭借生成优惠码。我们拥有优惠码，字典`$str_rand`。所以我们可以获取每次生成的伪随机数。  
  
exp.py
```python
str1='abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'
str2='SUjJQvy1e2NyihU'
str3 = str1[::-1]
length = len(str2)
res=''
for i in range(len(str2)):
	if i<=length/2:
		for j in range(len(str1)):
			if str2[i] == str1[j]:
				res+=str(j)+' '+str(j)+' '+'0'+' '+str(len(str1)-1)+' '
				break
	else:
		for j in range(len(str3)):
			if str2[i] == str1[j]:
				res+=str(len(str1)-j)+' '+str(len(str1)-j)+' '+'0'+' '+str(len(str1)-1)+' '
				break
print(res)
# 54 54 0 61 56 56 0 61 9 9 0 61 45 45 0 61 52 52 0 61 21 21 0 61 24 24 0 61 27 27 0 61 58 58 0 61 34 34 0 61 13 13 0 61 38 38 0 61 54 54 0 61 55 55 0 61 6 6 0 61 
```
使用`php_mt_seed`爆破：
```bash
php_mt_seed 54 54 0 61 56 56 0 61 9 9 0 61 45 45 0 61 52 52 0 61 21 21 0 61 24 24 0 61 27 27 0 61 58 58 0 61 34 34 0 61 13 13 0 61 38 38 0 61 54 54 0 61 55 55 0 61 6 6 0 61 
```
![](/images/19-7-9_PHP_rand_tool-for-use_4.png)