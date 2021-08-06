# 0x00 前言

因为M1芯片的限制，无法在本地安装GitBook环境，所以另辟蹊径，通过Docker安装环境。

# 0x01 制作Docker镜像

拉取一个nodejs的镜像

```bash
docker pull node:12.22.4-slim
```

运行镜像

```bash
docker run -v /Users/j7ur8/Share/WebBook/BookBuild:/Book -p4000:4000 -it  node:12.22.4-slim /bin/bash
```

更新软件

```bash
apt-get update
```

安装gitbook

```bash
npm install gitbook-cli@2.1.2 --global
cd ~/.gitbook/versions/3.2.3/node_modules/npm/
vim package.json 
# change the version of graceful-fs to 4.2.0,and then
npm install
```

# 0x02 使用

```bash
cd /book
gitbook build
```

