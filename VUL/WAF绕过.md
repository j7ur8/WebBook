参考：
- 先知的waf/upload_bypass

### 常见扩展黑名单
```
asp|asa|cer|cdx|aspx|ashx|ascx|asax
php|php2|php3|php4|php5|asis|htaccess
htm|html|shtml|pwml|phtml|phtm|js|jsp
vbs|asis|sh|reg|cgi|exe|dll|com|bat|pl|cfc|cfm|ini
```

### 多个Content-Disposition绕过

在IIS的环境下，上传文件时如果存在多个Content-Disposition的话，IIS会取第一个
Content-Disposition中的值作为接收参数，而如果waf只是取最后一个的话便会被绕过。

### 30个绕过方法

1. filename在content-type下面
2. .asp{80-90}
3. NTFS ADS
4. .asp...
5. boundary不一致
6. iis6分号截断asp.asp;asp.jpg
7. apache解析漏洞php.php.ddd
8. boundary和content-disposition中间插入换行
9. hello.php:a.jpg然后hello.<<<
10. filename=php.php
11. filename="a.txt";filename="a.php"
12. name=\n"file";filename="a.php"
13. content-disposition:\n
14. .htaccess文件
15. a.jpg.\nphp
16. 去掉content-disposition的form-data字段
17. php<5.3 单双引号截断特性
18. 删掉content-disposition: form-data;
19. content-disposition\00:
20. {char}+content-disposition
21. head头的content-type: tab
22. head头的content-type: multipart/form-DATA
23. filename后缀改为大写
24. head头的Content-Type: multipart/form-data;\n
25. .asp空格
26. .asp0x00.jpg截断
27. 双boundary
28. file\nname="php.php"
29. head头content-type空格:
30. form-data字段与name字段交换位置