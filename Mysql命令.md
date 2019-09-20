```
show databases; 
# 显示数据库
use mysql; 
# 选择操作默认数据库mysql ，以下操作默认表，皆是mysql
show tables; 
# 显示默认数据库中的表
show cloumns from user；
# 显示mysql.user表中的字段
select * from mysql.user;  
# 显示mysql.user表中的所有值
create table if not exist ceshi(
	`dbip` char(20),
	`name` char(20),
	`score` int auto_increment,
	primary key(`dbname`)
default charset=utf8;
# 如果表ceshi不存在，则创建，其内部字段为dbip，name，score，类型为char，char，和int型，其主键为score，并且其为自增长，设置其默认字符串为utf8
insert into ceshi(
dbip,name,score)
values
("192.168.1.1","HostA",18);
# 向表ceshi中插入数据为“192.168.1.1，HostA，18，并其对应字段为dbip，name，18
drop table ceshi; 
# 删除表ceshi
select * from user where user='root'; 
# 当表user中user字段为root的时候，显示所有值
select * from user where binary user='root'; 
# 当表user中user字段为root的时候，显示所有值,要求区分大小写

update mysql.user set password=password('centos') where user='hehe';
# 使用update修改数据库mysql中表user中字段password的值为centos，且当字段user为heh时，其密码centos使用password函数加密
delete mysql.user where user='5'; 
# 删除库mysql中表user中，当user字段为5的数据

select user,host,password from mysql.user where host like '127%';
# 显示库mysql中表user中字段user，host，password的值，且当host字段的值匹配127开头的时候
select user,host,password from mysql.user where user='root' and host like '127%';
# 显示库mysql中表user中字段user，host，password的值，且当user字段为root并且host字段匹配127开头的时候
	'%a'     //以a结尾的数据
	'a%'     //以a开头的数据
	'%a%'    //含有a的数据
	'_a_'    //三位且中间字母是a的
	'_a'     //两位且结尾字母是a的
	'a_'     //两位且开头字母是a的
# 如上，匹配条件的示例，%表示任意内容，_表示一个任意字符

selcet country,name from Webs where country='CN' union all select country, app_name Apps_country where country='CN' order by country;
# 查询表Web中country，name字段，且当country为CN时，同时也查询表Apps中app_name，App_conunty字段且当country为CN时，将内容都显示出来，并排序，重复项也要显示出来
# 单独使用union选项时候，是默认去掉重复项的，其中order by表示按照指定字段排序
select user,id,password,host from mysql.user order by id;
# 从库mysql中表user中查询字段user，id，password，host的值，并以字段ip排序显示


实例演示
mysql> SELECT * FROM employee_tbl;
+----+--------+---------------------+--------+
| id | name   | date                | singin |
+----+--------+---------------------+--------+
|  1 | 小明 | 2016-04-22 15:25:33 |      1 |
|  2 | 小王 | 2016-04-20 15:25:47 |      3 |
|  3 | 小丽 | 2016-04-19 15:26:02 |      2 |
|  4 | 小王 | 2016-04-07 15:26:14 |      4 |
|  5 | 小明 | 2016-04-11 15:26:40 |      4 |
|  6 | 小明 | 2016-04-04 15:26:54 |      2 |
+----+--------+---------------------+--------+
6 rows in set (0.00 sec)
# 示例表的数据
mysql> select name,count(*) from employee_tbl group by name;
+--------+----------+
| name   | count(*) |
+--------+----------+
| 小丽   |        1 |
| 小明   |        3 |
| 小王   |        2 |
+--------+----------+
3 rows in set (0.06 sec)
# group by表示按分组排序，而order表示按指定字段排序
mysql> select name,sum(id) from employee_tbl group by name;
+--------+---------+
| name   | sum(id) |
+--------+---------+
| 小丽   |       3 |
| 小明   |      12 |
| 小王   |       6 |
+--------+---------+
3 rows in set (0.07 sec)
mysql> select name,sum(singin) from employee_tbl group by name with rollup;
+--------+-------------+
| name   | sum(singin) |
+--------+-------------+
| 小丽   |           2 |
| 小明   |           7 |
| 小王   |           7 |
| NULL   |          16 |
+--------+-------------+
4 rows in set (0.00 sec)
# 可以使用sum，count等函数进行匹配
mysql> select coalesce(name,"总数"),sum(singin) from employee_tbl group by name with rollup;
+-------------------------+-------------+
| coalesce(name,"总数")   | sum(singin) |
+-------------------------+-------------+
| 小丽                    |           2 |
| 小明                    |           7 |
| 小王                    |           7 |
| 总数                    |          16 |
+-------------------------+-------------+
4 rows in set (0.03 sec)
# 我们可以使用 coalesce 来设置一个可以取代 NUll 的名称，coalesce（a,b,c) 语法：
# 如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null（没意义）。


select * from text.ceshi where score is null;
# 选出库test中表ceshi中的值，当score字段为空的时候，注意，null值只能使用is null和is not null来进行判断

# 查找name字段中以'st'为开头的所有数据：
mysql> SELECT name FROM person_tbl WHERE name REGEXP '^st';
# 查找name字段中包含'mar'字符串的所有数据：
mysql> SELECT name FROM person_tbl WHERE name REGEXP 'mar';
# 查找name字段中以元音字符开头或以'ok'字符串结尾的所有数据：
mysql> SELECT name FROM person_tbl WHERE name REGEXP '^[aeiou]|ok$';
# 在mysql语句中使用regexp来使用正则表达式


示例展示
mysql> select * from duiying;
+-------------+----------+----------+
| dbname      | username | password |
+-------------+----------+----------+
| 192.168.8.1 | HostA    | wenlong  |
+-------------+----------+----------+
1 row in set (0.00 sec)
# 显示表duiying原有数据
mysql> alter table duiying drop dbname;
Query OK, 1 row affected (0.04 sec)
Records: 1  Duplicates: 0  Warnings: 0
mysql> select * from duiying;
+----------+----------+
| username | password |
+----------+----------+
| HostA    | wenlong  |
+----------+----------+
1 row in set (0.00 sec)
# 使用alter和drop删除表duiying中的字段
mysql> alter table duiying add dbip int;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0
mysql> select * from duiying;
+----------+----------+------+
| username | password | dbip |
+----------+----------+------+
| HostA    | wenlong  | NULL |
+----------+----------+------+
1 row in set (0.00 sec)
# 使用alter和add增加表duiying中的字段
mysql> alter table duiying add dbname char(20) first;
Query OK, 1 row affected (0.03 sec)
Records: 1  Duplicates: 0  Warnings: 0
mysql> select * from duiying;
+--------+----------+----------+------+
| dbname | username | password | dbip |
+--------+----------+----------+------+
| NULL   | HostA    | wenlong  | NULL |
+--------+----------+----------+------+
1 row in set (0.00 sec)
# 在添加字段的时候可以使用first和after来调整字段在表中的位置

mysql> ALTER TABLE testalter_tbl RENAME TO alter_tbl;
# 数据表 testalter_tbl 重命名为 alter_tbl：
mysql> ALTER TABLE testalter_tbl MODIFY c CHAR(10);
# 把字段 c 的类型从 CHAR(1) 改为 CHAR(10)
mysql> ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
# 使用 ALTER 命令及 DROP子句来删除字段的默认值
mysql> ALTER TABLE testalter_tbl CHANGE j j INT;
# 使用 CHANGE 子句, 语法有很大的不同。 在 CHANGE 关键字之后，紧跟着的是你要修改的字段名，然后指定新字段名及类型

select database(); 
# 显示当前数据库
select user(); 
# 显示当前用户
show status; 
# 显示服务器zhuangt
select version(); 
# 显示mysql版本
show variables; 
# 显示服务器变量
```