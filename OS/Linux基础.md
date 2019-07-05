
## 概念解读

### 交互式和非交互式的区别在于
- 交互式shell等待你的输入，执行你提交的命令，返回你结果。  
- 非交互式，shell不与你进行实时交互，而是读取存放在文件仲的命令


## 基本命令

### Linux中查看所有正在运行的进程
参考：
- http://os.51cto.com/art/201101/244090.htm

```bash
ps -aux
top
```

### 查看端口开放情况
参考：
- https://blog.csdn.net/ws379374000/article/details/74218530
  
```bash
#查看当前所有tcp端口
netstat -ntlp   
#查看所有80端口使用情况
netstat -ntulp |grep 80   
```
### 文件解压

tar.gz
- `tar -zxvf xx.tar.gz`

tar.bz2
- `tar -jxvf xx.tar.bz2`



## 环境配置

### 阿里云软件源
```bash
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

### 多版本php安装
```bash
sudo apt install python-software-properties software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
```


### 添加环境变量
- https://www.cnblogs.com/joshua317/p/6899057.html

方法一：
```bash
export PATH=/usr/local/bin:$PATH
#配置完后可以通过echo $PATH查看配置结果。
#生效方法：立即生效
#有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢#复原有的path配置
#用户局限：仅对当前用户
```

方法二
```bash
#通过修改.bashrc文件:
vim ~/.bashrc 
#在最后一行添上：
export PATH=/usr/local/bin:$PATH
#生效方法：（有以下两种）
#1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
#2、输入“source ~/.bashrc”命令，立即生效
#有效期限：永久有效
#用户局限：仅对当前用户
```

方法三：
```bash
#通过修改profile文件:
vim /etc/profile
export PATH=/usr/local/bin:$PATH
#生效方法：系统重启
#有效期限：永久有效
#用户局限：对所有用户
```

方法四
```bash
#通过修改environment文件:
vim /etc/environment
在PATH="/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin"中加入 
":/usr/local/bin"
#生效方法：系统重启
#有效期限：永久有效
#用户局限：对所有用户
```


### 设置LD_LIBRARY_PATH
```bash
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

### sed替换源
```bash
sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
    sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list
```

### 配置ssh的root登录
```bash
sed -i 's/PermitRootLogin no/PermitRootLogin yes/' /etc/sshd/sshd_config
```

### 查看安装的所有包
```bash
dpkg -l
```


### sources.list和sources.list.d的区别
简单理解：
- sources.list是官方源或者，aliyun源这些。
- sources.list.d这个目录存放第三方软件的源，如php的ondrej源。

### wget10进制地址出错
情况：  
如果使用下列命令
```bash
wget 2130706433
#400 bad request
```

添加Host头部就好了
```bash
wget --headers='host: a.c' 2130706433
```