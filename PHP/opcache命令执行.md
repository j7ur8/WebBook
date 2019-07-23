# opcache命令执行

#### 参考

- http://www.vuln.cn/6763
- https://github.com/GoSecure/php7-opcache-override

#### 利用条件

- 文件上传漏洞，上传点需要可控或者为opcache.file_cache的目录。
- opcache.file_timestamps=0（默认为1）或者被缓存文件时间戳已知。
- opache.file_cache_only=1（默认为0）或者文件无内存缓存。因为内存缓存优先级别高于文件缓存

## 安装

```bash
apt-get install php7.0-opcache
sed -i 's~;opcache.file_cache=~opcache.file_cache=/tmp/opcache~' /etc/php/7.0/apache2/php.ini
sed -i 's~;opcache.validate_timestamps=1~opcache.validate_timestamps=0~' /etc/php/7.0/apache2/php.ini
sed -i 's~;opcache.file_cache_only=0~opcache.file_cache_only=1~' /etc/php/7.0/apache2/php.ini
service apache2 restart
```

### 利用过程

- 确认目标可能存在漏洞环境。
- 本地生成shell的opcache二进制文件shell.php.bin。
- 更改shell.php.bin的systemid，如果知道目标文件时间戳，也更改shell.php.bin的时间戳。
- 上传shell.php.bin到opcache缓存目录（如：/tmp/opcache/(systemid)/var/www/index.php.bin）
- 重新访问index.php

简单叙述如下：  
访问index.php，如果opcache_cache文件夹中存在该文件的index.php.bin缓存文件，则加载该文件，不访问网站根目录下的index.php。  

假如我们已知一个目标站点存在的文件，我们构造其对应的缓存文件放置到opcache_cache文件夹中，则该文件会被加载。  

但是访问缓存文件时，php会检测时间轴的一致性以及是否存在内存缓存。这也是我们为什么设置file_timestamps和file_cache_only的原因。  

但是我们如果已知文件时间轴并且该文件不常用，则我们可以伪造缓存文件的时间戳，绕过timestamps。而文件不常用，则不存在内存缓存。

