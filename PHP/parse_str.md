# parse_str

## parse_str

### 参考文章

- https://secure.php.net/manual/zh/function.parse-str.php
- https://github.com/hongriSec/PHP-Audit-Labs/blob/master/Part1/Day7/files/README.md

### 函数构造


```php
parse_str ( string $encoded_string [, array &$result ] ) : void
```

#### 版本变化

| 版本  | 说明                                                         |
| :---- | :----------------------------------------------------------- |
| 7.2.0 | 不带第二个参数的情况下使用 **parse_str()** 会产生 **E_DEPRECATED** 警告。 |

### 测试

parse_str的作用就是解析字符串并且注册成变量，但是在注册变量之前，他不会验证当前变量是否存在，所以会覆盖掉当前作用域中原有的变量。
![](../images/19-7-9_PHP_parse_str_1.png)

```php
<?php
$j7ur8='best';
parse_str('j7ur8=didi');
echo $j7ur8; 
# didi php<7.2
# PHP Deprecated:  parse_str(): Calling parse_str() without the result argument is deprecated       PHP7.2
```

## CTF题目

index.php
```php
<?php
$a = “hongri”;
$id = $_GET['id'];
@parse_str($id);
if ($a[0] != 'QNKCDZO' && md5($a[0]) == md5('QNKCDZO')) {
    echo '<a href="uploadsomething.php">flag is here</a>';
}
?>
```

uploadsomething.php
```php
<?php
header("Content-type:text/html;charset=utf-8");
$referer = $_SERVER['HTTP_REFERER'];
if(isset($referer)!== false) {
    $savepath = "uploads/" . sha1($_SERVER['REMOTE_ADDR']) . "/";
    if (!is_dir($savepath)) {
        $oldmask = umask(0);
        mkdir($savepath, 0777);
        umask($oldmask);
    }
    if ((@$_GET['filename']) && (@$_GET['content'])) {
        //$fp = fopen("$savepath".$_GET['filename'], 'w');
        $content = 'HRCTF{y0u_n4ed_f4st}   by:l1nk3r';
        file_put_contents("$savepath" . $_GET['filename'], $content);
        $msg = 'Flag is here,come on~ ' . $savepath . htmlspecialchars($_GET['filename']) . "";
        usleep(100000);
        $content = "Too slow!";
        file_put_contents("$savepath" . $_GET['filename'], $content);
    }
   print <<<EOT
<form action="" method="get">
<div class="form-group">
<label for="exampleInputEmail1">Filename</label>
<input type="text" class="form-control" name="filename" id="exampleInputEmail1" placeholder="Filename">
</div>
<div class="form-group">
<label for="exampleInputPassword1">Content</label>
<input type="text" class="form-control" name="content" id="exampleInputPassword1" placeholder="Contont">
</div>
<button type="submit" class="btn btn-default">Submit</button>
</form>
EOT;
}
else{
    echo 'you can not see this page';
}
?>
```