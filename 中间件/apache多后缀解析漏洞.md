### apache多后缀解析漏洞
Apache HTTPD 支持一个文件拥有多个后缀，并为不同后缀执行不同的指令。比如，如下配置文件：  
```bash
AddType text/html .html
AddLanguage zh-CN .cn
```
  
其给`.html`后缀增加了media-type，值为`text/html`；给`.cn`后缀增加了语言，值为`zh-CN`。此时，如果用户请求文件`index.cn.html`，他将返回一个中文的html页面。  

以上就是Apache多后缀的特性。如果运维人员给`.php`后缀增加了处理器：  

```
AddHandler application/x-httpd-php .php
```

那么，在有多个后缀的情况下，只要一个文件含有`.php`后缀的文件即将被识别成PHP文件，没必要是最后一个后缀。利用这个特性，将会造成一个可以绕过上传白名单的解析漏洞。  
![](/images/apache多后缀解析漏洞1.png)
![](/images/apache多后缀解析漏洞2.png)

### apache解析特性
`apache1.x~2.x`  
Apache在以上版本中，解析文件名的方式是从后向前识别扩展名，直到遇见Apache可识别的扩展名为止。