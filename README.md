# Introduction
对于web方面漏洞的学习和记录
有一些还没有记载，尽自己能力和心情做到详细吧

目录大致如下：
<pre>
├─语言
│  ├─PHP
│  │  ├─内核错误
│  │  │  └─PHP7的2个core dumped错误
│  │  ├─语法特性
│  │  │  ├─弱比较
│  │  │  ├─PHP7和PHP5的区别
│  │  │  ├─数组的数组和二维数组
│  │  │  └─深入理解$_REQUEST数组
│  │  ├─内置函数、类的利用
│  │  │  ├─内置函数和类
│  │  │  ├─利用类进行XXE攻击
│  │  │  │  └─SimpleXMLElement
│  │  │  ├─利用类进行反序列化
│  │  │  │  └─Phar
│  │  │  ├─利用类进行CRLF攻击
│  │  │  │  └─Soapclient
│  │  │  ├─通过反射调用类
│  │  │  │  └─重构函数和反射
│  │  ├─函数使用不当
│  │  │  ├─getimagesize
│  │  │  ├─in_array
│  │  │  ├─filter_var
│  │  │  ├─strpos
│  │  │  ├─escapeshellarg和escapeshellcmd
│  │  │  ├─parse_str
│  │  │  ├─preg_replace
│  │  │  ├─unserialize
│  │  │  ├─htmlentities
│  │  │  ├─rand
│  │  │  ├─extract
│  │  │  ├─create_function
│  │  │  ├─parse_url
│  │  │  └─code-breaking_easy-phplimit函数的巧妙搭配
│  │  ├─正则
│  │  │  └─pcre回溯问题
│  │  ├─Session安全问题
│  │  │  ├─包含session文件
│  │  │  ├─Session的不同解析方式
│  │  │  └─自定义Session处理函数
│  │  ├─文件上传与文件包含
│  │  │  ├─PHP伪协议
│  │  │  ├─包含日志
│  │  │  ├─包含environ
│  │  │  ├─包含临时文件
│  │  │  └─包含session文件
│  │  ├─Bypass disable function
│  │  │  ├─攻击后端组
│  │  │  │  ├─opcache命令执行
│  │  │  │  ├─gd库漏洞
│  │  │  │  ├─魔图漏洞
│  │  │  │  └─破壳漏洞
│  │  │  ├─mod_cgi模式
│  │  │  ├─寻找未禁用的漏网函数。
│  │  │  └─LD_PRELOAD劫持系统函数
│  │  └─PHP框架
│  │     └─YII
│  │        └─YII框架全版本文件包含漏洞挖掘和分析
│  ├─Python
│  │  ├─危险函数
│  │  │  ├─eval
│  │  │  └─input等
│  │  ├─格式化字符串漏洞
│  │  │  ├─f修饰符与任意代码执行
│  │  │  ├─获取泄露Django配置信息
│  │  │  └─Jinja 2.8.1 模板沙盒绕过
│  │  ├─SSTI
│  │  │  ├─Tornado
│  │  │  └─Flask
│  │  ├─沙箱逃逸
│  │  ├─反序列化
│  │  │  ├─Numpy反序列化命令执行(CVE-2019-6446)浅析
│  │  │  ├─pickle反序列化
│  │  │  └─PyYAML反序列化
│  │  └─库漏洞
│  │     ├─PIL命令执行
│  │     │  ├─CVE-2017-8291
│  │     │  └─CVE-2018-16509
│  │     ├─http.server重定向
│  │     └─urllib-CRL注入
│  │        ├─CVE-2016-5699
│  │        └─CVE-2019-9740
│  ├─Java
│  │  ├─JNDI注入
│  │  ├─RMI反序列化
│  │  └─Spring
│  │     └─Spring Boot Actuators
│  ├─Go
│  │  ├─特性
│  │  │  └─slice切片问题
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
│  │  │  ├─UDF提权
│  │  │  ├─MOF方法提权
│  │  │  ├─启动项提权
│  │  │  ├─MYyQL-CVE提权
│  │  │  └─Mysql数据库反弹端口连接提权
│  │  ├─SQL注入
│  │  │  ├─SQL注入研究
│  │  │  └─SQL注入研究总结篇
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
│  │  │  ├─缓存投毒和缓存欺骗
│  │  │  └─http/0.9进行缓存投毒
│  │  ├─RPO
│  │  │  └─RPO学习
│  │  └─XSS
│  │      └─XSS学习
│  └─JWT
│     └─jwt伪造
├─中间件
│  ├─Apache
│  │  └─换行解析漏洞
│  ├─Elasticsearch
│  │  ├─CVE-2014-3120
│  │  ├─CVE-2015-1427
│  │  ├─CVE-2015-3337
│  │  └─CVE-2015-5531
│  ├─Fastcgi
│  │  └─未授权访问及任意命令执行
│  ├─Nginx
│  │  ├─Nginx文件名逻辑漏洞
│  │  ├─Nginx越界读取缓存漏洞
│  │  └─Nginx配置错误CVE-2017-10271
│  ├─IIS
│  │  ├─IIS短文件名
│  │  ├─IIS命令执行漏洞
│  │  └─IIS6.0文件解析漏洞
│  ├─weblogic
│  │  ├─XMLDecoder反序列化漏洞
│  │  ├─Components反序列化命令执行漏洞
│  │  ├─Weblogic任意文件上传漏洞
│  │  ├─SSRF漏洞
│  │  └─Weblogic常规渗透测试环境
│  └─tomcat
│     ├─信息泄露
│     └─任意代码执行
└─操作系统
   ├─Windows
   │  ├─软件漏洞
   │  │  └─winrar目录穿越漏洞
   │  └─提权
   ├─Linux
   │  └─提权
   │     └─脏牛提权
   └─MAC
      └─提权
         └─IOHIDFamily提权
</pre>