```bash
docker run -it -p9000:80 ubuntu:20.04 /bin/bash

apt update

apt install vim apache2 php libapache2-mod-php
#选 6 和 70
apt install mysql-server mysql-client -y

# 添加mysql的密码
service mysql start

mysql #回车

mysql> use mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'lamplamp';
mysql> exit
```

