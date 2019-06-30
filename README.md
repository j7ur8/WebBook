# 更新

## 2019.6

### 6.30
增加了`PHP/匿名函数.md、PHP/回调函数.md`  
还有一些小更新可能没有记载，以及一些近期学习的将会在7月份更新。

### 6.16
完善了·SQL/PDO真伪预处理分析.md·。

### 6.25
增加了Code/Python.md，Code/HTML.md。添加了一些常用的代码。

### 6.18
1. 完善了部分中间件的解析漏洞文章(文件上传相关)，增加了VUL/文件上传大类并基本完善收集到的姿势。
2. 增加了`PHP/源码编译安装php和apache`
3. 调整了readme.md的格式

### 6.12
1. 完善了php/pcre回溯问题、php/包含session文件、php/包含临时文件
2. 更新了Docker基本操作
3. 增加了php/源码编译安装php和apache

### 6.9
1. 增加了'xxe基础payload'
2. 添加了堆叠注入的历史以及如何关闭多语句执行。
3. 添加了'mime_content_type.md'

### 6.3
1. 增加了mysql中like的用法的说明以及盲注的姿势，盲注的内容更加丰富了;
2. 增加了PHP下的文件上传绕过文章。
4. 增加了PHP的open_basedir绕过文章
5. 增加OS下'windwos基础'、'Linux基础'和'Docker基本操作'
6. php7下2个core dumped错误增加了利用版本范围。

## 2019.5

### 5.30
更新了python脚本编写基础，反序列化漏洞基础，以及堆叠注入。



# Introduction

对于web方面漏洞的学习和记录  
有一些还没有记载，尽自己能力和心态做到详细吧  

目录大致如下：（更多看请看这里）(https://j7ur8.github.io/WebBook/)
<pre>
├─语言
│  ├─PHP
│  │  ├─内核错误
│  │  ├─语法特性
│  │  ├─内置函数、类的利用
│  │  │  ├─内置函数和类
│  │  │  ├─利用类进行XXE攻击
│  │  │  │  └─SimpleXMLElement
│  │  │  ├─利用类进行反序列化
│  │  │  │  └─Phar
│  │  │  ├─利用类进行CRLF攻击
│  │  │  │  └─Soapclient
│  │  │  └─通过反射调用类
│  │  │     └─重构函数和反射
│  │  ├─函数使用不当
│  │  ├─正则
│  │  ├─Session安全问题
│  │  ├─文件上传与文件包含
│  │  ├─Bypass disable function
│  │  └─PHP框架
│  │     └─YII
│  ├─Python
│  │  ├─危险函数
│  │  ├─格式化字符串漏洞
│  │  ├─SSTI
│  │  │  ├─Tornado
│  │  │  └─Flask
│  │  ├─沙箱逃逸
│  │  ├─反序列化
│  │  └─库漏洞
│  │     ├─PIL命令执行
│  │     ├─http.server重定向
│  │     └─urllib-CRL注入
│  ├─Java
│  │  ├─JNDI注入
│  │  ├─RMI反序列化
│  │  └─Spring
│  ├─Go
│  │  ├─特性
│  │  └─Go语言任意代码执行漏洞 CVE-2018-6574
│  └─JavaScript
│     ├─Nodejs
│     │  └─原型注入
│     └─相关库漏洞
│        └─angularjs模板注入
├─数据库
│  ├─PDO
│  ├─MySQL
│  │  ├─MYSQL渗透工具
│  │  ├─MySQL提权
│  │  ├─SQL注入
│  │  ├─PhpMyadmin各版本漏洞
│  │  ├─MywebSQL漏洞
│  │  └─mysql任意文件读取漏洞
│  ├─MSSQL
│  │  ├─提权
│  │  └─MSSQL注入
│  ├─NoSQL
│  │  └─NoSQL注入
│  ├─Redis
│  │  └─未授权访问漏洞
│  └─SQLite
│     └─SQLite注入
├─HTTP相关
│  ├─HTTP
│  │  ├─缓存投毒和缓存欺骗
│  │  ├─RPO
│  │  └─XSS
│  └─JWT
├─中间件
│  ├─Apache
│  ├─Elasticsearch
│  ├─Fastcgi
│  ├─Nginx
│  ├─IIS
│  ├─weblogic
│  └─tomcat
├─操作系统
│  ├─Windows
│  │  ├─软件漏洞
│  │  └─提权
│  ├─Linux
│  │  └─提权
│  └─MAC
│     └─提权
└─漏洞类型
   ├─XXE
   └─反序列化
</pre>

