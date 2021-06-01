### Ubuntu18.04安装MySQL后普通用户无法登陆
参考：
- https://blog.csdn.net/bingjianit/article/details/82780535
  
原因：
```sql
select user, plugin from mysql.user;

+------------------+-----------------------+
| user             | plugin                |
+------------------+-----------------------+
| root             | auth_socket           |
| mysql.session    | mysql_native_password |
| mysql.sys        | mysql_native_password |
| debian-sys-maint | mysql_native_password |
+------------------+-----------------------+

```
  
解决：
```sql
update mysql.user set authentication_string=PASSWORD('1234'), plugin='mysql_native_password' where user='root';

flush privileges;
```