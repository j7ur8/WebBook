### 一个不完善的安装php7.2.10版本的apache+php环境。
```docker
FROM ubuntu:18.04

RUN sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
    sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
    apt-get update

# 安装必须的工具
RUN apt-get install --no-install-recommends -y apt-utils gcc build-essential make libexpat1-dev libxml2-dev wget

#apr、apr-units、pcre、apache、php
RUN wget -P /root http://mirror.reverse.net/pub/apache/apr/apr-1.6.5.tar.bz2  --no-check-certificate
RUN wget -P /root http://mirror.reverse.net/pub/apache//apr/apr-util-1.6.1.tar.gz  --no-check-certificate
RUN wget -P /root https://ftp.pcre.org/pub/pcre/pcre-8.38.tar.gz  --no-check-certificate
RUN wget -P /root http://apache.mirrors.hoobly.com//httpd/httpd-2.4.39.tar.bz2  --no-check-certificate
RUN wget -P /root https://www.php.net/distributions/php-7.2.10.tar.bz2  --no-check-certificate

# 解压
RUN tar zxvf /root/apr-util-1.6.1.tar.gz -C /root
RUN tar zxvf /root/pcre-8.38.tar.gz -C /root
RUN tar jxvf /root/apr-1.6.5.tar.bz2 -C /root
RUN tar jxvf /root/httpd-2.4.39.tar.bz2 -C /root
RUN tar jxvf /root/php-7.2.10.tar.bz2 -C /root

# 安装pcre
RUN cd /root/pcre-8.38
RUN ./configure && make && make install

# 安装apache2
RUN mv /root/apr-1.6.5 /root/httpd-2.4.39/srclib/apr
RUN mv /root/apr-util-1.6.1 /root/httpd-2.4.39/srclib/apr-util
RUN cd /root/httpd-2.4.39
RUN ./configure --with-pcre=/usr/local/bin/pcre-config --with-included-apr && make && make install 
RUN export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

# 安装php7210
RUN cd /root/php-7.2.10
RUN ./configure --prefix=/usr/local/php7210 --with-apxs2=/usr/local/apache2/bin/apxs && make && make install
RUN /usr/local/apache2/build/libtool --finish /root/php-7.2.10/libs
RUN cp /root/php.ini-development /usr/local/php7210/lib/php.ini

# 配置httpconfig和php
RUN sed -i 's/DirectoryIndex index.html/DirectoryIndex index.html index.php/g' /usr/local/apache2/conf/httpd.conf
RUN echo 'U2VydmVyTmFtZSBsb2NhbGhvc3QKCjxGaWxlc01hdGNoIFwucGhwJD4KICAgIFNldEhhbmRsZXIgYXBwbGljYXRpb24veC1odHRwZC1waHAKPC9GaWxlc01hdGNoPg==' | base64 -d >> /usr/local/apache2/conf/httpd.conf

CMD /usr/local/apache2/bin/apachectl -k start & /bin/bash -c "while true;do echo hello docker;sleep 1;done"
```

说不完善是因为当你直接在bash中使用上述命令，那么没问题，但是如果你使用`docker buildt -t`来创建并`run`这个镜像，访问映射的端口连接时，会出现拒绝连接，`exec`进入容器内部，会发现apache报错:
`Apache is running a threaded MPM, but your PHP Module is not compiled to be threadsafe. You need to recompile PHP.`