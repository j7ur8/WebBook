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

### 文件解压

tar.gz
- `tar -zxvf xx.tar.gz`

tar.bz2
- `tar -jxvf xx.tar.bz2`


### 环境变量
- https://www.cnblogs.com/joshua317/p/6899057.html

### 设置LD_LIBRARY_PATH
```bash
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

### sed替换源
```bash
sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
    sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list
```