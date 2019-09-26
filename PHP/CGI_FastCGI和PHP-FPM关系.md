# CGI_FastCGI和PHP-FPM的关西

参考：
- https://www.awaimai.com/371.html
- https://www.zhihu.com/question/20153311
  
先理解静态和伪静态。
在一个网站架构中，WebServer（如Apache）被称为中间件，是服务端和客户端之间沟通的媒介。当服务端请求一个静态页面时，一般WebServer会去文件系统中找到这个文件，并直接传递给浏览器。
![](/images/19-7-4_PHP_FashCGI和PHP-FMP的关系1.png)
  
如果请求的是伪静态文件，根据配置信息（如.htaccess),WebServer知道这个不是静态文件，需要PHP解析器来处理，那么WebServer会把这个请求根据自身配置进行一定处理，然后交给PHP解析器。
![](/images/19-7-4_PHP_FashCGI和PHP-FMP的关系2.png)
  
当WebServer收到index.php请求后，会启动对应的CGI程序（也就是上文的PHP的解析器）。接下来PHP解析器解析配置文件（php.ini），初始化执行环境，处理请求，然后以CGI规定的格式返回WebServer处理后的结果，退出进程。最后WebServer再把结果返回给浏览器。
  
以上就是一个完整的动态PHP Web访问流程。由此，我们可以介绍以下概念：
- WebServer：Apache，Nginx、IIS、Lighttpd、Tomcat等服务器
- WebApplication：PHP、Python、Java开发的应用程序。
- CGI：全称：Common Gateway Interface。是WebServer与WebApplication之间数据交换的一种协议。
- FastCGI：同CGI，是一种通信协议，单笔CGI在效率上做了一些优化。同样，SCGI协议与FastCGI类似。
- PHP-CGI：是PHP（WebApplication）对WebServer提供的CGI协议接口程序。
- PHP-FPM：是PHP（WebApplication）对WebServer提供的FastCGI协议的接口程序，额外还提供了相对智能的任务管理。