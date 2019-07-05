### 自己建的容器
PHP:
- j7ur8/lamp:7.0

### docker容器run后不退出的办法
参考：
- https://blog.csdn.net/guizaijianchic/article/details/78208088
  
```bash
docker run -d centos /bin/bash -c "while true;do echo hello docker;sleep 1;done"
```

### dockerfile的CMD命令

```bash
CMD service apache2 start & tail -F /var/log/apache2/access.log
```

### 替换源
```dockerfile
RUN sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list &&\
	sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list
```
