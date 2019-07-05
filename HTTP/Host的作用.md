参考：
- https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html#0x04-nginx-host

Phit0on师傅的解答：

> 众所周知，如果我们在浏览器里输入`http://2018.mhz.pw`，浏览器将先请求DNS服务器，获取到目标服务器的IP地址，之后的TCP通信将和域名没有关系。那么，如果一个服务器上有多个网站，那么Nginx在接收到HTTP包后，将如何区分？  
>  
> 这就是Host的作用：用来区分用户访问的究竟是哪个网站（在Nginx中就是Server块）。 