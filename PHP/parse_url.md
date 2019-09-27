# parse_url

- https://skysec.top/2017/12/15/parse-url%E5%87%BD%E6%95%B0%E5%B0%8F%E8%AE%B0/
- https://skysec.top/2018/03/15/Some%20trick%20in%20ssrf%20and%20unserialize()/#trick2-libcurl-and-parse-url
- https://www.php.net/ChangeLog-5.php

## parse_url

### 错误的解析hostname

#### 参考

- https://bugs.php.net/bug.php?id=73192
  

#### 利用范围

- PHP5.x < php5.6.28
- PHP7.0 < php7.0.13

#### 测试代码

**demo.php**

```php
<?php
echo parse_url("http://example.com:80#@google.com/")["host"]."\n"; #google.com
echo parse_url("http://example.com:80?@google.com/")["host"]."\n"; #google.com
echo file_get_contents("http://example.com:80#@google.com");       # example.com 
?>
```

### 不解析有端口但没有协议网址

#### 参考

- https://bugs.php.net/bug.php?id=68917
  

#### 利用范围

- PHP5.6 < PHP5.6.8
- PHP5.5 < PHP5.5.24
- PHP5.4.x

#### 测试代码

**demo.php**

```php
<?php
print_r(parse_url('//example.org:81/hi?a=b#c=d'));   // 
print_r(parse_url('//example.org/hi?a=b#c=d'));   //Array([host] => example.org [path] => /hi [query] => a=b [fragment] => c=d )
```

### 无法解析空用户名和密码

#### 参考

- https://bugs.php.net/bug.php?id=68129

#### 利用范围

- PHP5.6 < PHP5.6.3
- PHP5.5 < PHP5.5.19
- PHP5.4.x

#### 测试代码

**demo.php**

```php
<?php
// correct (returns empty username)
var_dump( parse_url( 'https://@example.com' ) );
// incorrect (doesn't return empty username or password)
var_dump( parse_url( 'https://:@example.com' ) );
// incorrect (doesn't return empty username)
var_dump( parse_url( 'https://:password@example.com' ) );
// incorrect (doesn't return empty password)
var_dump( parse_url( 'https://username:@example.com' ) );
```

### 对于//的解析错误
#### 参考

- https://bugs.php.net/bug.php?id=62844
- https://skysec.top/2017/12/15/parse-url%E5%87%BD%E6%95%B0%E5%B0%8F%E8%AE%B0/#%E5%89%8D%E8%AE%B0

#### 利用范围

- php5.x < php5.4.7

#### 测试代码

**demo.php**

```php
<?php
var_dump(parse_url('//example.org'));  # array(1) {'path' => string(14) "//example.org"}
var_dump(parse_url('http:/xxx.com/abc//def?g=1&e=2'))['path'] // def?g=1&e=2
var_dump(parse_url('http:/xxx.com/abc///def?g=1&e=2'))['path'] // FALSE
```

### curl和parse_url的解析顺序问题

#### 利用范围

- 全版本
- libcurl 7.52，windows下的curl-7.55.1版本会报错`curl: (3) Port number ended with '@'`

#### 测试代码

demo.php

```php
<?php
var_dump(parse_url('http://u:p@a.com:80@b.com/'));  //host: b.com
system('curl http://u:p@a.com:80@b.com/')  //couldn't resolve host a.com
```