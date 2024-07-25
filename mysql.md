# MySQL数据库

## 数据分类

##### 1、数据按照结构类型分为：结构型数据，半结构化数据，非结构化数据。

​	（1）结构化数据一般是指可以用二维表来逻辑表达实现的数据。是有固定的格式和有限长度的数据，

可以用 关系型数据库表示和存储。二维表

​    （2）半结构化数据：HTML文档，JSON，XML和一些 NoSQL 数据库等就属于半结构化数据。

​	（3)   非结构化数据包括：二进制文件，音视频文件，位置信息等

##### 2、数据库（DB） 数据库管理系统（DBMS） 数据库管理员（DBA）

##### 3、数据库分类

（1）层次数据库：仅限于一对多的关系。类似于树；

（2）网状数据库：可以一对多，可以多对多，类似于图；

（3）用了关系模型来组织数据的数据库，其以行和列的形式存储数据，类似于表；一般工作在c/s模式

​		其中：主键是一个或多个字段的组合，用于惟一确定一个记录的字段，一张表只有一个主键， 主键字段不能为空 NULL。

​		唯一键：一个或多个字段的组合，用于惟一确定一个记录的字段，一张表可以有多个字段可以为NULL。

##### 4、第三范式：第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。

##### 5、MySQL 安装

Rocky 8.6/9.4 安装 

```bash
安装 mysql-server，会自动安装客户端包
[root@rocky86 ~]# yum install -y mysql-server
..........
#服务状态
[root@rocky86 ~]# systemctl status mysqld.service
...........
#启动服务
[root@rocky86 ~]# systemctl enable --now mysqld.service
#自动创建的账户
[root@rocky86 ~]# getent passwd mysql
#查看家目录
[root@rocky86 ~]# ll /var/lib/mysql
LISTEN     0      50           *:3306 *:* users:

```

##### 6、客户端连接

mysql或mysql -uroot -p

![image-20240724195427118](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724195427118.png)



##### 7、设置密码



##### ![image-20240724200259940](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724200259940.png)

修改密码可用 `mysql -uroot -p‘123456’ password '654321';`

##### 8、mysql常见命令

？  查看所有客户端命令

`![image-20240724204112826](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724204112826.png)

help contents  查看所有服务端命令        help+命令名称，查看命令具体项

![](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724204324337.png)

- 显示版本号 `mysql -V`
- 指定用户名，主机，端口 `mysql -uroot -h127.0.0.1 -p3306 -p123456`
- 查看数据库列表：`show databases`
- 切换数据库：`use student`
- 调用系统命令 `\! + 命令`

![image-20240724210747582](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724210747582.png)

- 修改提示符`prompt[\h--\D]`主机--时间

  在文件中修改提示符`vim /etc/my.conf.d/client.cnf`

  ​     

![image-20240724211847013](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724211847013.png)

- 重连  `\r`

- tee 文件名    创建一个文件，文件中保存输入输出结果    \t  停止保存

![image-20240724212348040](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724212348040.png)



![image-20240724212645516](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724212645516.png)

-  `select version()\g`发送命令到服务端

- `select version()\p` 只输出不执行命令

  

##### 9、mysqladmin 命令，在系统执行

- `mysqladmin  -V 或mysqladmin  version`  查看版本号

- `mysqladmin status` 显示状态信息

- 设置超时连接

  ![image-20240724213755305](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724213755305.png)

  

- `mysqladmin -i 1 ping`持续执行ping命令

- `mysqladmin -i 1 -c 3 ping` 执行三次ping命令

- `mysqladmin shutdown`关闭mysql服务，mysqladmin不能开启mysql服务

- `mysqladmin create/drop student`创建/删除 student数据库 `mysql -uroot -p123456 -e 'show databases'` 查看数据库

  `mysqladmin -f drop student1`强制删除数据库

  ![image-20240724214816117](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240724214816117.png)

  

  ```bash
  - `mysqladmin -uroot -p123456 -hlocalhost password 13579`修改密码
  ```

  

##### 11、下载mycli

```
[root@m52 ~]# yum install python38

......

#使用国内源

[root@m52 ~]# pip3.8 config set global.index-url 
https://pypi.tuna.tsinghua.edu.cn/simple

[root@m52 ~]# pip-3.8 install mycli

......

[root@m52 ~]# mycli -uroot -p13579
```

##### 12、sql语言

1. 关键字不区分大小写，具体对象区分大小写，以；结尾
2. 一个单词之间不能跨行
3. 在 .sql 文件中，注释       --  单行注释   /*    */   多行注释  mysql中注释用#
4. 数据库对象命名以字母开头，符号只能用 #_$    不能使用mysql保留字

###### sql语句·

- 查看字符集：`show chartacter set /  show charset `   MySQL 8.0 开始默认字符集为 utf8mb4，也可在/etc/my.cnf.d/client.cnf修改，重启服务生效。

-  查看保存的数据库文件`ls /var/lib/mysql`

- `show create database student1`  查看数据库创建语句

-  `create database if not exists student`如果不存在该数据库，就创建

- 创建数据库时指定字符集`create database student3 DEFAULT CHARACTER SET latin1;`

  

![image-20240725093043764](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725093043764.png)

- 修改字符集`ALTER DATABASE student character set utf8mb4 COLLATE  utf8mb4_0900_ai_ci;`

- 删除数据库 `drop database student3  /   drop database if exists student3`

###### 数据类型：数值，日期，字符串

- 数值型： 整数型 int    浮点数型（近似值） float(位数,小数点位数)    double(位数，小数点位数)     定点数：（存放精确值）                                declimal(数字位数，小数点位数)

- 日期 DATETIME（8字节）、DATE（3）、TIMESTAMP（4）、TIME（3）和YEAR（1）。

- 字符串char(n) 和 varchar(n) 中括号中 n 代表字符的个数，存储和检索方式不同

  ##### DDL语句（主要用来操作数据库中的表。）

- 创建表

  

```sql
use student

CREATE TABLE class (
id int UNSIGNED AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(20) NOT NULL,
age tinyint UNSIGNED,
#height DECIMAL(5,2),
gender ENUM('M','F') default 'M'
)ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4;
```

![image-20240725101555569](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725101555569.png)



- 复制表（复制现存的表的表结构创建，但不复制数据）

![image-20240725102209402](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725102209402.png)

- `show create table student;/show create table student \G;`查看建表语句

- `show engines \G;` 查看引擎

- `alter table student rename class` 修改表名

- `ALTER TABLE stu ADD phone varchar(11) AFTER name;`在name后加一列phone

- ![image-20240725110417669](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725110417669.png)

- `ALTER TABLE stu MODIFY phone int;`修改字段类型

- `ALTER TABLE stu CHANGE COLUMN phone mobile char(11)`修改字段名称和类型 

  alter table 表名 change column 列名 改后列名 条件

-  `ALTER TABLE class DROP COLUMN mobile;`删除字段

- ``ALTER TABLE class character SET utf8;``修改字符集

- 同时修改数据类型和字符集`ALTER TABLE class CHANGE name name char(30) charactersetutf8;`

- `alter table class alter column gender set default 'M';`将gender默认字段改为M

-  `ALTER TABLE class ADD PRIMARY KEY (id);`添加主键

-  `ALTER TABLE class DROP PRIMARY KEY;`删除主键

- `drop table class;drop table student.class3;`删除表

##### DML语句 （insert，update，delete）

- 插入数据/插入多条数据

  ![image-20240725113912917](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725113912917.png)

  

- 更新数据：一定要加条件限制，没有条件则会更新表中的所有记录。

![image-20240725114503632](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725114503632.png)

- `delete from class where id=10` 删除数据（物理删除）
- 逻辑删除：加一列is_del  当is_del为1时 代表删除

![image-20240725115400628](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725115400628.png)

- 清空表 `truncate TABLE class;`

##### 13、DQL语句

- select 查字符串，制定别名，数值计算，查询变量

![image-20240725143338213](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725143338213.png)

- 查询指定字段

  ![image-20240725144224931](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725144224931.png)

- 查询姓名为空的信息`select * from claass where age  IS  NULL/ age  IS  NOT  NULL`

- 模糊匹配 以x开头 /   结尾的名字 `select id,name,age from class where name like 'x%'   /    '%x';`

  

![image-20240725145127413](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240725145127413.png)

- 区间过滤

  ```sql
  #过滤id在2到五之间的学生姓名
  select id,name from class where id  between 2 and 5;
  #过滤id在2到五之外的学生姓名
  select id,name from class where id not between 2 and 5;
  #过滤出id为2,3,5的学生
  select id,name from class where id  in (2,3,5);
  #过滤出id不是2,3,5的学生
  select id,name from class where id not in (2,3,5);
  ```

- 统计和分组

  

```sql
#统计表有多少条，用total表示
(root@localhost) [student]>select count(*) as total from class;
+-------+
| total |
+-------+
|     5 |
+-------+
1 row in set (0.00 sec)
#统计age列有多少条（NULL不统计）
(root@localhost) [student]>select count(*) as total from class;
+-------+
| total |
+-------+
|     5 |
+-------+
1 row in set (0.00 sec)
#取，最大最小平均值
(root@localhost) [student]>select max(id),min(id),avg(age) from class;
+---------+---------+----------+
| max(id) | min(id) | avg(age) |
+---------+---------+----------+
|       5 |       1 |  17.7500 |
+---------+---------+----------+
1 row in set (0.00 sec)
#求和
(root@localhost) [student]>select sum(age) from class where id between 1 and 4;
+----------+
| sum(age) |
+----------+
|       71 |
+----------+
1 row in set (0.00 sec)
#分组
root@localhost) [student]>select count(*),max(id),sum(age),avg(age),gender from class where age is not null group by gender;
+----------+---------+----------+----------+--------+
| count(*) | max(id) | sum(age) | avg(age) | gender |
+----------+---------+----------+----------+--------+
|        2 |       3 |       34 |  17.0000 | M      |
|        3 |       4 |       57 |  19.0000 | F      |
+----------+---------+----------+----------+--------+
2 rows in set (0.00 sec)
(root@localhost) [student]>select count(*) as total,max(id),sum(age),avg(age),gender from class where age is not null group by gender having total=3;
+-------+---------+----------+----------+--------+
| total | max(id) | sum(age) | avg(age) | gender |
+-------+---------+----------+----------+--------+
|     3 |       5 |       55 |  18.3333 | M      |
+-------+---------+----------+----------+--------+
1 row in set (0.00 sec)
```

- 排序

```sql
#按照id正向排序
(root@localhost) [student]>select id,name,age from class order by id;
(root@localhost) [student]>select id,name,age from class order by id asc;
#按照id倒序
(root@localhost) [student]>select id,name,age from class order by id desc;
#在排序对象前加负号，排序结果相反
(root@localhost) [student]>select id,name,age from class order by -id desc; #升序
```

- 去重

```sql
(root@localhost) [student]>select distinct age from class order by age desc;   #将年龄倒序去重

(root@localhost) [student]>select distinct(age) from class order by age desc;    #将年龄倒序去重
```

- 分页

```
#limit 0,3 从1开始，长度为3的分页
(root@localhost) [student]>select id,name,age from class limit 0,3;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   16 |
|  2 | 小红   |   17 |
|  3 | 小刚   |   18 |
+----+--------+------+
3 rows in set (0.00 sec)
#从第5个开始，长度为3
(root@localhost) [student]>select id,name,age from class limit 4,3;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  5 | 狗蛋   |   21 |
|  6 | 铁柱   |   21 |
|  7 | 小刚   |   18 |
+----+--------+------+
3 rows in set (0.00 sec)
```

##### 14、多表操作

```sql
#过滤class表中年龄小于teacher年龄平均值-20的
(root@localhost) [student]>select name,age from class where age<(select avg(age)-20 from teacher);
#将teacher表中最小年龄-20更新到class表中id=5的年龄上
(root@localhost) [student]>update class set age=(select min(age)-20 from teacher) where id=5;
#用于in的子查询 筛选出比class中比teacher小20的age
(root@localhost) [student]>select id,name,age from class where age in (select age-20 from teacher);
#exist如果是true 输出符号括号条件内容，也可not exist（+条件）
(root@localhost) [student]>select id,name from class where exists (select id from teacher where class.id=teacher.id);


```

- 多表操作

  

```sql
#两个表联合一块union，如果字段不同合并容易列名可能不正确
(root@localhost) [student]>select id,name,age from class union select id,name,age from teacher;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   16 |
|  2 | 小红   |   17 |
|  3 | 小刚   |   18 |
|  4 | 翠花   |   20 |
|  5 | 狗蛋   |   20 |
|  6 | 铁柱   |   21 |
|  7 | 小刚   |   18 |
|  8 | 66     | NULL |
|  1 | wang   |   40 |
|  2 | zhang  |   45 |
|  3 | li     |   50 |
|  4 | cui    |   40 |
|  5 | lv     |   45 |
+----+--------+------+
13 rows in set (0.00 sec)
#合并，默认去重
select * from teacher union select * from teacher;
#合并，不去重 union all 
select * from teacher union all select * from teacher;

```

- 交叉连接，

  ```sql
  (root@localhost) [student]>select * from class
      -> ;
  +----+--------+------+--------+--------+
  | id | name   | age  | gender | is_del |
  +----+--------+------+--------+--------+
  |  1 | 小明   |   16 | M      |      0 |
  |  2 | 小红   |   17 | F      |      0 |
  |  3 | 小刚   |   18 | M      |      0 |
  |  4 | 翠花   |   20 | F      |      0 |
  +----+--------+------+--------+--------+
  4 rows in set (0.00 sec)
  
  (root@localhost) [student]>select * from teacher
      -> ;
  +----+------+------+--------+
  | id | name | age  | gender |
  +----+------+------+--------+
  |  3 | li   |   50 | M      |
  |  4 | cui  |   40 | M      |
  |  5 | lv   |   45 | M      |
  +----+------+------+--------+
  3 rows in set (0.00 sec)
  #交叉连接 cross join
  (root@localhost) [student]>select * from class cross join teacher;
  +----+--------+------+--------+--------+----+------+------+--------+
  | id | name   | age  | gender | is_del | id | name | age  | gender |
  +----+--------+------+--------+--------+----+------+------+--------+
  |  1 | 小明   |   16 | M      |      0 |  5 | lv   |   45 | M      |
  |  1 | 小明   |   16 | M      |      0 |  4 | cui  |   40 | M      |
  |  1 | 小明   |   16 | M      |      0 |  3 | li   |   50 | M      |
  |  2 | 小红   |   17 | F      |      0 |  5 | lv   |   45 | M      |
  |  2 | 小红   |   17 | F      |      0 |  4 | cui  |   40 | M      |
  |  2 | 小红   |   17 | F      |      0 |  3 | li   |   50 | M      |
  |  3 | 小刚   |   18 | M      |      0 |  5 | lv   |   45 | M      |
  |  3 | 小刚   |   18 | M      |      0 |  4 | cui  |   40 | M      |
  |  3 | 小刚   |   18 | M      |      0 |  3 | li   |   50 | M      |
  |  4 | 翠花   |   20 | F      |      0 |  5 | lv   |   45 | M      |
  |  4 | 翠花   |   20 | F      |      0 |  4 | cui  |   40 | M      |
  |  4 | 翠花   |   20 | F      |      0 |  3 | li   |   50 | M      |
  +----+--------+------+--------+--------+----+------+------+--------+
  12 rows in set (0.00 sec)
  
  ```

  

- 内连接取交集

  ```sql
  #取两个表id相同的并集
  (root@localhost) [student]>select * from class inner join teacher on class.id=teacher.id;
  +----+--------+------+--------+--------+----+------+------+--------+
  | id | name   | age  | gender | is_del | id | name | age  | gender |
  +----+--------+------+--------+--------+----+------+------+--------+
  |  3 | 小刚   |   18 | M      |      0 |  3 | li   |   50 | M      |
  |  4 | 翠花   |   20 | F      |      0 |  4 | cui  |   40 | M      |
  +----+--------+------+--------+--------+----+------+------+--------+
  ```

- 自然连接

  

```
select * from t1;select * from t2;
+----+------+
| id | name |
+----+------+
|  1 | zhao |
|  2 | wang |
|  3 | li   |
+----+------+
3 rows in set (0.001 sec)
+----+-------+
| id | title |
+----+-------+
|  1 | cto   |
|  2 | ceo   |
|  3 | cfo   |
+----+-------+
3 rows in set (0.000 sec)
#自然连接

MariaDB [db1]> select * from t1 NATURAL JOIN t2;
+----+------+-------+
| id | name | title |
+----+------+-------+
|  1 | zhao | cto   |
|  2 | wang | ceo   |
|  3 | li   | cfo   |
+----+------+-------+

```

- sql注入

  ```sql
  #将where条件改为真
  (root@localhost) [student]>select * from user;
  +----+-------+----------+
  | id | user  | password |
  +----+-------+----------+
  |  2 | user  | 654321   |
  |  1 | admin | 123456   |
  +----+-------+----------+
  2 rows (root@localhost) [student]>select * from user where user='admin' and password='123456';
  +----+-------+----------+
  | id | user  | password |
  +----+-------+----------+
  |  1 | admin | 123456   |
  +----+-------+----------+
  1 row in set (0.00 sec)
  
  (root@localhost) [student]>select * from user where user='admin' and password='def' or 1=1;
  +----+-------+----------+
  | id | user  | password |
  +----+-------+----------+
  |  2 | user  | 654321   |
  |  1 | admin | 123456   |
  +----+-------+----------+
  2 rows in set (0.00 sec)
   set (0.00 sec)
  ```

##### 15、view视图



```sql
#创建view视图
root@localhost) [student]>create view v_stu as select * from class where age>20;
Query OK, 0 rows affected (0.00 sec)

(root@localhost) [student]>show tables;
+-------------------+
| Tables_in_student |
+-------------------+
| class             |
| teacher           |
| user              |
| v_stu             |
+-------------------+
4 rows in set (0.01 sec)
#当表class发生变化时，视图 v_stu 也会发生变化
#查询和更新与表相同，更新后若不满足视图的条件（age<20）则不在v_stu表里
#v_stu视图中数据变化，class表跟着变化
#删除视图
drop view v_stu;

```

##### 16、function

```sql
#mysql8.0创建自定义函数报错
(root@localhost) [student]>CREATE FUNCTION simpleFun() RETURNS VARCHAR(20) RETURN "Hello World";
ERROR 1418 (HY000): ...............
(root@localhost) [student]> select @@log_bin_trust_function_creators;
+-----------------------------------+
| @@log_bin_trust_function_creators |
+-----------------------------------+
|                                 0 |
+-----------------------------------+
1 row in set, 1 warning (0.00 sec)
#把@@log_bin_trust_function打开
(root@localhost) [student]>set global log_bin_trust_function_creators=ON;
Query OK, 0 rows affected, 1 warning (0.00 sec)

(root@localhost) [student]> select @@log_bin_trust_function_creators;
+-----------------------------------+
| @@log_bin_trust_function_creators |
+-----------------------------------+
|                                 1 |
+-----------------------------------+
1 row in set, 1 warning (0.00 sec)
#创建成功
(root@localhost) [student]>CREATE FUNCTION simpleFun() RETURNS VARCHAR(20) RETURN "Hello World";
Query OK, 0 rows affected (0.01 sec)
(root@localhost) [student]>select simplefun();
+-------------+
| simplefun() |
+-------------+
| Hello World |
+-------------+
1 row in set (0.00 sec)
```

##### 17、MySQL 用户管理

- Host 可以写成 主机名，IP地址，网段。可以用 %，_ 来表示通配符，% 表示任意长度的任意字符，_ 表 示一个字符。

  

```sql
#无法在远程主机上连接当前主机的MySQL服务

[root@m52 ~]# mysql -uroot -h10.0.0.164 -p
Enter password: 
ERROR 1130 (HY000): Host '10.0.0.177' is not allowed to connect to this MySQL 
server
#创建用户
create user test@'10.0.0.0/255.255.255.0' identified by '123456';
create user test2@'10.0.0.%' identified by '123456';
#用户重命名
RENAME USER 'USERNAME'@'HOST' TO 'USERNAME'@'HOST';
#删除用户
ROP USER 'USERNAME'@'HOST'

#范例，删除默认的空用户
DROP USER ''@'localhost';
#修改密码
SET PASSWORD FOR root@'localhost'='123456' ;

[root@localhost ~]# mysqladmin -uroot -p123456 password '654321'

```

```sql
#MySQL5.7 和 8.0 破解 root 密码
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
skip-grant-tables
skip-networking #MySQL8.0不需要
[root@localhost ~]# systemctl restart mysqld
[root@localhost ~]# mysql
#方法1
mysql> update mysql.user set authentication_string='' where user='root' and 
host='localhost';
#方法2
mysql> flush privileges;
#再执行下面任意一个命令
mysql> alter user root@'localhost' identified by 'ubuntu';
mysql> set password for root@'localhost'='ubuntu';
[root@localhost ~]# vim /etc/my.cnf

[mysqld]
#skip-grant-tables
#skip-networking
[root@localhost ~]# systemctl restart mysqld
[root@localhost ~]# mysql -uroot -pubuntu
```

