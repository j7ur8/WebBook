参考：
- https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html#0x03-filter_validate_email

### host解析

**对Host的解析**   
Ngnix处理Host时，以冒号分割Host为Hosname和port两部分，port部分会被丢弃掉。  
  
**Ngnix与PHP-FPM对Host的解析顺序差异**  
在以下2个Host头中
```http
Host: 2018.mhz.pw
Host: xxx'"@example.com
```
Ngnix认为Host时`2018.mhz.pw`;   
PHP中使用`$_SERVER['HTTP_HOST']`取到的值却是`xxx'"@example.com`  