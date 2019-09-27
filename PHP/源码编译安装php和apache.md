# 源码编译安装php和apache

## Liunx环境下

具体的命令可以参看以下的命令，不进行具体解释。

```docker
sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
    sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
    apt-get update

# 安装必须的工具
apt-get install --no-install-recommends -y apt-utils gcc build-essential make libexpat1-dev libxml2-dev wget

#apr、apr-units、pcre、apache、php
wget -P /root http://mirror.reverse.net/pub/apache/apr/apr-1.6.5.tar.bz2  --no-check-certificate
wget -P /root http://mirror.reverse.net/pub/apache//apr/apr-util-1.6.1.tar.gz  --no-check-certificate
wget -P /root https://ftp.pcre.org/pub/pcre/pcre-8.38.tar.gz  --no-check-certificate
wget -P /root http://apache.mirrors.hoobly.com//httpd/httpd-2.4.39.tar.bz2  --no-check-certificate
wget -P /root https://www.php.net/distributions/php-7.2.10.tar.bz2  --no-check-certificate

# 解压
tar zxvf /root/apr-util-1.6.1.tar.gz -C /root
tar zxvf /root/pcre-8.38.tar.gz -C /root
tar jxvf /root/apr-1.6.5.tar.bz2 -C /root
tar jxvf /root/httpd-2.4.39.tar.bz2 -C /root
tar jxvf /root/php-7.2.10.tar.bz2 -C /root

# 安装pcre
cd /root/pcre-8.38
./configure && make && make install

# 安装apache2
mv /root/apr-1.6.5 /root/httpd-2.4.39/srclib/apr
mv /root/apr-util-1.6.1 /root/httpd-2.4.39/srclib/apr-util
cd /root/httpd-2.4.39
./configure --with-pcre=/usr/local/bin/pcre-config --with-included-apr && make && make install 
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

# 安装php7210
cd /root/php-7.2.10
./configure --prefix=/usr/local/php7210 --with-apxs2=/usr/local/apache2/bin/apxs && make && make install
/usr/local/apache2/build/libtool --finish /root/php-7.2.10/libs
cp /root/php.ini-development /usr/local/php7210/lib/php.ini

# 配置httpconfig和php
sed -i 's/DirectoryIndex index.html/DirectoryIndex index.html index.php/g' /usr/local/apache2/conf/httpd.conf
echo 'U2VydmVyTmFtZSBsb2NhbGhvc3QKCjxGaWxlc01hdGNoIFwucGhwJD4KICAgIFNldEhhbmRsZXIgYXBwbGljYXRpb24veC1odHRwZC1waHAKPC9GaWxlc01hdGNoPg==' | base64 -d >> /usr/local/apache2/conf/httpd.conf

```

## Windows环境下

没有相关需求，官方貌似给出了一些参考文章。

- https://wiki.php.net/internals/windows/stepbystepbuild_sdk_2
- https://wiki.php.net/internals/windows/stepbystepbuild



