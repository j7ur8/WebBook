### docker容器run后不退出的办法
参考：
- https://blog.csdn.net/guizaijianchic/article/details/78208088
  
```bash
docker run -d centos /bin/bash -c "while true;do echo hello docker;sleep 1;done"
```