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



##### 18、权限



```sql
#给wordpresser库所有权限
GRANT ALL ON wordpress.* TO wordpress@'10.0.0.%';
#授予所有权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.0.0.%' WITH GRANT OPTION;
#取消权限
REVOKE DELETE ON *.* FROM 'testuser'@'172.16.0.%'
REVOKE ALL ON *.* FROM 'testuser'@'172.16.0.%';
#查看指定用户权限
SHOW GRANTS FOR 'user'@'host';
#查看当前使用中的用户的权限
SHOW GRANTS FOR CURRENT_USER[()];
```

```sql
#创建用户
(root@localhost) [(none)]>create user 'root'@'10.0.0.%' identified by '123456';
Query OK, 0 rows affected (1.89 sec)
#查看用户列表
(root@localhost) [(none)]>select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | 10.0.0.%  |
| wordpresser      | 10.0.0.%  |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
6 rows in set (0.00 sec)
#查看用户权限，新用户只有usage权限，只能连接
(root@localhost) [(none)]>show grants for 'root'@'10.0.0.%';
+-----------------------------------------+
| Grants for root@10.0.0.%                |
+-----------------------------------------+
| GRANT USAGE ON *.* TO `root`@`10.0.0.%` |
+-----------------------------------------+
1 row in set (0.00 sec)
#远程连接
mysql -uroot -p123456 -h10.0.0.164
#授权只能查看 student数据库的class表
(root@localhost) [(none)]>grant select on student.class to root@'10.0.0.%';
Query OK, 0 rows affected (0.30 sec)

(root@localhost) [(none)]>show grants for 'root'@'10.0.0.%';
+--------------------------------------------------------+
| Grants for root@10.0.0.%                               |
+--------------------------------------------------------+
| GRANT USAGE ON *.* TO `root`@`10.0.0.%`                |
| GRANT SELECT ON `student`.`class` TO `root`@`10.0.0.%` |
+--------------------------------------------------------+
2 rows in set (0.00 sec)
```

##### 19、MySQL架构和性能优化

```
#查看最大并发连接数
(root@localhost) [(none)]>show variables like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.00 sec)
```

- 存储引擎

-  mysql5.5之后用InooDB 

   mysql5.5之前用MylSAM：读写相互阻塞 不能同时进行

- 锁：计算机协调多进程或线程并发访问某一资源时的机制

  页锁

  表锁：对整张表加锁 ，MylSAM   开销小，加锁块，不会出现死锁

  行锁：只对当前的操作行加锁   IonoDB 冲突概率低，开销大，加锁慢，会出现死锁

```
#查看引擎
(root@localhost) [student]>show table status like 'user'\G;
*************************** 1. row ***************************
           Name: user
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 2
 Avg_row_length: 8192
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 3
    Create_time: 2024-07-25 17:22:01
    Update_time: 2024-07-25 17:24:46
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.01 sec)
#查看表文件
[root@10 ~]$ ll /var/lib/mysql
```

- MVCC : 多版本并发控制，可以看做行级锁的一个变种

```sql
#查看 mysql 支持的存储引擎
show engines;
#查看默认的存储引擎
show variables like '%storage_engine%';
#在文件中设置存储引擎
vim /etc/my.cnf
[mysqld]
default_storage_engine=InnoDB
#查看student库中所有表使用的存储引擎
show table status from student \G;
#查看特定表默认存储引擎
show create table class \G;
show table status like 'class'\G;
#设置表的存储引擎
CREATE TABLE class(... ) ENGINE=InnoDB;
ALTER TABLE class ENGINE=InnoDB;
```

- ##### 服务器选项

- 查看服务器选项`mysqld --verbose --help`

- 主配置文件

```bash
#在文件查看服务器选项
[root@localhost ~]# cat /etc/my.cnf.d/mysql-server.cnf 
...... ...... 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysql/mysqld.log
pid-file=/run/mysqld/mysqld.pid
#加服务器选项报错查看错误信息
tail /var/log/mysql/mysqld.log
```

- ##### 服务器系统变量

- ```sql
  #查看所有变量/所有全局变量/session变量
  (root@localhost) [(none)]>show variables;
  (root@localhost) [(none)]>show global variables;
  (root@localhost) [(none)]>show session variables;#修改会话变量：仅对当前会话有影响
  #查看具体变量
  (root@localhost) [(none)]>show variables like '变量名';
  (root@localhost) [(none)]>select @@变量名;
  #变量无法实现永久保存，重启服务后会被重置
  
  ```

##### 20、index索引

- 数据结构：快速查找 

- 索引通过存储引擎实现

- 优点

​		大大加快数据的检索速度;

​		创建唯一性索引，保证数据库表中每一行数据的唯一性;

​		加速表和表之间的连接;

​		在使用分组和排序子句进行数据检索时，可以显著减少查询中分组和排序的时间。

- 缺点

​		索引需要占物理空间。 

​		当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，降低了数据的维护速度。

-  树：n个节点，n-1条边，没有环路

   节点的度：子节点的个数

   树的度：最大节点的度就是树的度

  深度：根节点到节点唯一路径长

  高度：节点到树叶最长路径长

- 二叉查找树：首先它是一颗二叉树，若左子树不空，则左子树上所有结点的值均小于它的根结点的 值，若右子树不空，则右子树上所有结点的值均大于它的根结点的值，左、右子树也分别为二叉排 序树

  **红黑树特点** 

  - 根节点是黑色的，叶节点是不存储数据的黑色空节点
  -  任何相邻的两个节点不能同时为红色，红色节点被黑色节点隔开，红色节点的子节点是黑色的 
  - 任意节点到其可到达的叶节点间包含相同数量的黑色节点，保证任何路径相差不会超出2倍，从而 实现基本平衡

-   **B-树：读做B树**

     B树中是将数据和索引放在一起的，以减少IO次数， 加快查询速度，一个节点能放多少数据，通常取决于一条数据占用的空间大小。

![image-20240727093150330](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240727093150330.png)

- **B+树**

![image-20240727093236983](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240727093236983.png)

- **B+树和B-树的主要区别**：

B-树内部节点是保存数据的，而B+树内部节点是不保存数据的，只作索引作用，它的叶子节点才保 存数据。

B+树相邻的叶子节点之间是通过链表指针连起来的，B-树却不是。 查找过程中，B-树在找到具体的数值以后就结束，而B+树则需要通过索引找到叶子结点中的数据才 结束

B-树中任何一个关键字出现且只出现在一个结点中，而B+树可以出现多次。

- InnoDB 引擎中一颗的 B+ 树可以存放多少行数据？

总记录数=根节点指针数*单个叶子记录的行数

```sql
#查看student数据库，class表的索引
(root@localhost) [(none)]>show index from student.class;
#查看语句是否利用索引
(root@localhost) [student]>explain select * from class where id=4 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: class
   partitions: NULL
         type: const            #只读取一次，ALL全表扫描
possible_keys: PRIMARY
          key: PRIMARY          #使用了主键索引
      key_len: 4
          ref: const
         rows: 1                #扫描了一条数据
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
#创建索引
(root@localhost) [student]>create index idx_name on class(name);
#create index idx_name on student(name(10)); 表示取 name 字段中的前 10 个字符做索引
#查询姓名，从原来扫描六条数据降到扫描1条数据
root@localhost) [student]>explain select * from class where name='翠花'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: class
   partitions: NULL
         type: ref
possible_keys: idx_name         #使用了索引
          key: idx_name         #使用了索引
      key_len: 82
          ref: const
         rows: 1                #扫描1条数据，比全表扫描扫描的数据少
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.01 sec)
#B+树索引是左前缀特性，可以使用like查询左匹配'%x'，右匹配不能索引 'x%'
 explain select * from student where name like 'g%'\G;
#mariadb中查询索引次数
select count(*) from class where name like 'm%';
#删除索引
(root@localhost) [student]>drop index idx_name on class;
```

- profile工具

```sql
#查看profiling
(root@localhost) [student]>select @@profiling; 
#开启
(root@localhost) [student]>set profiling=1;
#运行一条命令
(root@localhost) [student]>select * from class;
.......
#查看
(root@localhost) [student]>show profiles;
+----------+------------+---------------------+
| Query_ID | Duration   | Query               |
+----------+------------+---------------------+
|        1 | 0.00084250 | select * from class |
+----------+------------+---------------------+
#duration 用时
#导入数据库
mysql 数据库名 < testlog.sql
#执行存储过程
mysql> call sp_testlog;

```

##### 21、锁

隐式锁； 由存储引擎自动施加锁 

显式锁； 用户手动请求

```sql
#给class表加读锁 不影响查询，无法写
(root@localhost) [student]>lock tables class read;
#释放锁
(root@localhost) [student]>unlock tables;
#连接或进程只能释放其自身施加的锁
#全局锁
#加读锁，可读不可写#不能创建账号，数据库，创建账号本质也是写表
 flush tables with read lock;

```

##### 22、事务

- 事务在InooDB下，mylsam不支持事务

- 显式启动事务`begin    begin work     start   transaction`

- 提交执行 commiit           回滚  rollback

  

```sql
#查看数据库的表
(root@localhost) [student]>select * from class where id=5;
+----+--------+------+--------+--------+
| id | name   | age  | gender | is_del |
+----+--------+------+--------+--------+
|  5 | 小张   |   21 | M      |      0 |
+----+--------+------+--------+--------+
1 row in set (0.00 sec)
#开始事务begin
(root@localhost) [student]>begin;
Query OK, 0 rows affected (0.00 sec)
#将id=5的学生年龄改为30
(root@localhost) [student]>update class set age='30' where id='5';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
#在本设备上查看class数据发生改变，但在另一设备访问本数据库，数据不变age=20
(root@localhost) [student]>select * from class where id='5';
+----+--------+------+--------+--------+
| id | name   | age  | gender | is_del |
+----+--------+------+--------+--------+
|  5 | 小张   |   30 | M      |      0 |
+----+--------+------+--------+--------+
1 row in set (0.00 sec)
#提交事务commit
(root@localhost) [student]>commit;
Query OK, 0 rows affected (0.01 sec)
#提交完之后，在其他设备发现class表中 age=30
```

回滚操作

```sql
#事务开始
root@localhost) [student]>begin;
Query OK, 0 rows affected (0.00 sec)
#删除表
(root@localhost) [student]>delete from class;
Query OK, 7 rows affected (0.15 sec)
#查询class表，如果在另一台设备查看数据库，class表有数据
(root@localhost) [student]>select * from class;
Empty set (0.00 sec)
#回滚
(root@localhost) [student]>rollback;
Query OK, 0 rows affected (0.11 sec)
#查看class表，回滚
(root@localhost) [student]>select * from class;
+----+--------+------+--------+--------+
| id | name   | age  | gender | is_del |
+----+--------+------+--------+--------+
|  1 | 小明   |   16 | M      |      0 |
|  2 | 小红   |   17 | F      |      0 |
|  3 | 小刚   |   18 | M      |      0 |
|  4 | 翠花   |   20 | F      |      0 |
|  5 | 小张   |   30 | M      |      0 |
|  6 | 小吕   |   20 | F      |      0 |
|  7 | 铁柱   |   23 | M      |      0 |
+----+--------+------+--------+--------+
7 rows in set (0.00 sec)
#truncate table class; 清空表， truncate 属于 DDL 语句，不支持事务操作，回滚数据并未回来
#drop 也不是DML语句，所以也无法回滚
#如果输入多条指令可以在中间加入事务保存点 mysql > savepoint p1   mysql > savepoint p2  ......
#需要回滚时，只需要回滚到对应保存点 mysql> rollback to p2  




#查看自动提交
(root@localhost) [student]>select @@autocommit;
+--------------+
| @@autocommit |
+--------------+
|            1 |
+--------------+
1 row in set (0.99 sec)
#atuocommit 是会话级别的变量，同时也是一个服务器选项，需要此选项永久生效，可以写配置文件
[root@10 /etc/my.cnf.d]$ vim mysql-server.cnf 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysql/mysqld.log
pid-file=/run/mysqld/mysqld.pid
autocommit=0
#重启后生效

```

- 查看正在进行的事务`SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;`

- 死锁：指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象， MySQL 中出现死锁时，MySQL 服务会自行处理，自行回滚一个事务，防止死锁的出现。

- \#MySQL8.0.13 及以后使用此语句查看锁`SELECT * FROM performance_schema.data_locks;`

  发生死锁时，需要kill +进程id  杀掉未完成的进程id

- MySQL 中默认的隔离级 别是可重复读。

![image-20240727164215339](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240727164215339.png)

mysql8.0

查看隔离级别：`select @@transaction_isolation;`

指定事务隔离级别`SET transaction_isolation='READ-UNCOMMITTED|READCOMMITTED|REPEATABLEREAD|SERIALIZABLE`

\#隔离级修改为 读未提交

`[root@rocky86 ~]# vim /etc/my.cnf.d/mysql-server.cnf` 

`[mysqld]`

`transaction-isolation=READ-UNCOMMITTED`

##### 23、日志

![image-20240727173129406](C:\Users\26914\AppData\Roaming\Typora\typora-user-images\image-20240727173129406.png)

```sql
#MySQL 通过 innodb_flush_log_at_trx_commit 配置来控制 redo log 的落盘规则。
#值为0：延迟写，先写到log buffer 每隔1s批量写os buffer和log file
#1：实时写 每次提交事务后都会将数据写入 OS Buffer 再写磁盘文件，这种方式几乎不会丢失数据，性能较差
#2： 实时写，延迟落盘 实时写入os buffer 写磁盘1s一次
(root@localhost) [(none)]>show variables like '%innodb_flush_log_at_trx%';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.00 sec)
#将值改为0
root@localhost) [student]>set global innodb_flush_log_at_trx_commit=0;
Query OK, 0 rows affected (0.00 sec)



#错误日志
#查看错误日志位置
(root@localhost) [student]>select @@log_error;
+---------------------------+
| @@log_error               |
+---------------------------+
| /var/log/mysql/mysqld.log |
+---------------------------+
1 row in set (0.00 sec)
#可在配置文件中修改路径
[root@10 ~]$ cat /etc/my.cnf.d/mysql-server.cnf 
[mysqld]
log-error=/var/log/mysql/mysqld.log
#错误日志记录基本 1：error   2：error warning 3：error warning information
mysql> select @@log_error_verbosity;
-----------------------+
| @@log_error_verbosity |
+-----------------------+
|                     2 |
+-----------------------+

```

**通用日志**

```sql
#相关配置项
#general_log=0|1 是否开启通用日志
#general_log_file=/path/log_file 日志文件
#log_output=FILE|TABLE|NONE 记录到文件中还是记录到表中
#查看默认配置
mysql> select @@general_log,@@general_log_file,@@log_output;

#开启日志
mysql> set global general_log=1;
Query OK, 0 rows affected (0.00 sec)
#修改参数，将日志记录到表中
mysql> select * from mysql.general_log;
Empty set (0.00 sec)
#修改配置
mysql> set global log_output="TABLE";
Query OK, 0 rows affected (0.00 sec)
mysql> select @@log_output;
+--------------+
| @@log_output |
+--------------+
| TABLE        |
+--------------+
1 row in set (0.00 sec)

```

**慢查询日志**

```sql
#相关配置项
#slow_query_log=0|1 是否开启记录慢查询日志
#log_slow_queries=0|1 同上，MariaDB 10.0/MySQL 5.6.1 版本后已删除
#slow_query_log_file=/path/log_file 日志文件路径
#long_query_time=N 对慢查询的定义，默认10S，也就是说一条SQL查询语
句，执行时长超过10S，将会被记录
#log_queries_not_using_indexes=0|1 只要查询语句中没有用到索引，或是全表扫描，都会记
录，不管查询时长是否达到阀值
#查看默认配置，默认慢查询没开
mysql> select @@slow_query_log,@@slow_query_log_file,@@long_query_time;
#开启慢查询日志，日志存储在slow_query_log_file下
mysql> set global slow_query_log=1;
Query OK, 0 rows affected (0.00 sec)
mysql> set long_query_time=1;
Query OK, 0 rows affected (0.00 sec)
#mysqldumpslow 查询慢排序文件
[root@10 ~]$ mysqldumpslow /var/lib/mysql/10-slow.log 

```

**二进制日志**

- 三种格式：

  statement：记录原生执行SQL语句，写什么记什么

  row：将sql语句变量函数替换后记录

  mixed：混合

```sql
#查询二进制日志信息
(root@localhost) [student]>select @@log_bin,@@sql_log_bin,@@log_bin_basename,@@log_bin_index,@@binlog_format,@@max_binlog_size\G
*************************** 1. row ***************************
         @@log_bin: 1        #是否开启二进制日志
     @@sql_log_bin: 1         #是否开启二进制日志
@@log_bin_basename: /var/lib/mysql/binlog            #binlog文件前缀
   @@log_bin_index: /var/lib/mysql/binlog.index      #索引文件路径
   @@binlog_format: ROW                              #row格式记录
 @@max_binlog_size: 1073741824                       #单文件大小
1 row in set, 1 warning (0.00 sec)
#MySQL中默认开启，查看相关文件，不是文本文件，无法查看
[root@rocky86 ~]# ll -h /var/lib/mysql/binlog.*

#查看正在使用的二进制日志
(root@localhost) [(none)]>show master logs;  #show master status 查看正在使用的二进制文件
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| 10-bin.000001 |       469 | No        |
+---------------+-----------+-----------+
1 row in set (0.00 sec)

(root@localhost) [(none)]>show binary logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| 10-bin.000001 |       469 | No        |
+---------------+-----------+-----------+
1 row in set (0.00 sec)
#查看二进制日志中的事件
(root@localhost) [(none)]>show binlog events
#重启或命令刷新会自动生成新的二进制文件
#[root@rocky86 ~]# mysqladmin flush-logs/[root@rocky86 ~]#systemctl restart mysql.service/mysql> flush logs;

(root@localhost) [(none)]>show binary logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| 10-bin.000001 |       469 | No        |
+---------------+-----------+-----------+
1 row in set (0.00 sec)

(root@localhost) [(none)]>flush logs;#刷新
Query OK, 0 rows affected (0.01 sec)
(root@localhost) [(none)]>show binary logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| 10-bin.000001 |       513 | No        |
| 10-bin.000002 |       157 | No        |
+---------------+-----------+-----------+
2 rows in set (0.00 sec)
#删除10-bin.000002之前的日志
mysql> PURGE BINARY LOGS TO 'binlog.000002';
#删除 2024年7月28号之前的日志
mysql> PURGE BINARY LOGS BEFORE '2023-01-01 21:00:00
#设置从01开始
mysql> reset master;

```

##### 24、备份恢复

1. mysqldump 备份和还原实现 -A备份所有，-B备份结构

   

```sql
#备份数据库student(此种备份方式并不会备份数据库结构，只备份表和数据)
#创建存放sql文件的文件夹
[root@10 ~]$ mkdir data/
#将想要备份的数据库student备份为student-bak.sql文件
[root@10 ~]$ mysqldump -uroot -p123456 student > /root/data/student-bak.sql
#测试，删除数据库的表
mysql> show tables;
+-------------------+
| Tables_in_student |
+-------------------+
| class             |
| teacher           |
| user              |
| v_stu             |
+-------------------+
4 rows in set (0.00 sec)

mysql> drop table class;
Query OK, 0 rows affected (0.01 sec)

mysql> drop table teacher;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+-------------------+
| Tables_in_student |
+-------------------+
| user              |
| v_stu             |
+-------------------+
2 rows in set (0.01 sec)
#备份还原 （也可重新创建数据库和源数据库不同名，数据还原到新数据库）
[root@10 ~]$ mysql -uroot -p123456 student < /root/data/student-bak.sql
#登录查看
mysql> show tables;
+-------------------+
| Tables_in_student |
+-------------------+
| class             |
| teacher           |
| user              |
| v_stu             |
+-------------------+
4 rows in set (0.00 sec)
```

```sql
#备份数据库student结构和数据
#将student数据库备份（备份文件中含有创建数据库语句，比单纯备份数据的文件大）
[root@10 ~]$ mysqldump -uroot -p123456 -B student > /root/data/student2-bak.sql
#测试，删除student数据库
mysql> drop database student;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| student2           |
| sys                |
+--------------------+
#还原数据库
[root@10 ~]$ mysql -uroot -p123456 < /root/data/student2-bak.sql 
#查看
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| student            |
| student2           |
| sys                |
+--------------------+
6 rows in set (0.00 sec)
```

2、备份所有数据库

```sql
#备份所有数据库
[root@10 ~]$ mysqldump -uroot -p123456 -A > /root/data/all-bak.sql
#备份数据库，生成压缩文件
mysqldump -uroot -p123456 -A | gzip >/root/data/all-bak.sql.gz
#查看备份了哪些数据库
[root@10 ~]$ cat /root/data/all-bak.sql | grep -i '^create database'
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `mysql` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `student` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `student2` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
#information_schema，performance_schema，sys 这三个数据库没有备份

##如果我们使用了全部备份，而又只需要恢复一个数据库或一个表时，则应该先将备份数据还原到一个中间数据库，再从该库备份出想要的数据，然后再进行还原操作。

```

3、数据库备份脚本

```bash
[root@rocky86 0104]# vim backup.sh

UNAME=root
PWD=12345
HOST=10.0.0.158
IGNORE='Database|information_schema|performance_schema|sys'
YMD=`date +%F`
if [ ! -d /backup/${YMD} ];then
 mkdir -pv /backup/${YMD}
fi
DBLIST=`mysql -u${UNAME} -p${PWD} -h${HOST} -e "show databases;" 2>/dev/null | 
grep -Ewv "$IGNORE"`
for db in ${DBLIST};do
 mysqldump -u${UNAME} -p${PWD} -h${HOST} -B $db 2>/dev/null 
1>/backup/${YMD}/${db}_${YMD}.sql
 if [ $? -eq 0 ];then
 echo "${db}_${YMD} backup success"
 else
 echo "${db}_${YMD} backup fail"
 fi
done
#测试
[root@rocky86 0104]# chmod a+x backup.sh

[root@rocky86 0104]# ls /backup/
#执行备份
[root@rocky86 0104]#./backup.sh 
#还原
[root@rocky86 0104]# mysql -uuser1 -p123456 < /backup/testdb_2023-01-04.sql
......
```

2、xtrabackup 备份和还原实现（先下载xtrabackup 插件）

```sql
#开始备份

[root@rocky86 ~]# xtrabackup -uuser1 -p123456 --backup --target-dir=/backup/base
......
......
[root@rocky86 ~]# ls /backup/base/
备份时的相关信息
[root@rocky86 ~]# cat /backup/base/xtrabackup_info
#远程主机还原
#远程主机需要安装相同版本的xtrabackup，相同版本的MySQL
#整理文件
[root@rocky86 ~]# xtrabackup --prepare --target-dir=/root/backup/base
#远程主机mysql服务不能开启，数据目录/var/lib/mysql/不能有数据
#还原
[root@rocky86 ~]# xtrabackup --copy-back --target-dir=/root/backup/base --datadir=/var/lib/mysql
#修改权限
[root@rocky86 ~]# chown -R mysql.mysql /var/lib/mysql/*
#启动查看
```

数据还原时注意，还原顺序一定要正确，先还原完全备份的数据，再还原第一次增量备份的数据，再还 原第二次增量备份的数据，如果有多个增量备份，也是按照此规则进行还原。另外，在还原时，只有最 后一次的备份文件还原时需要进行事务回滚，之前的都不用回滚。

##### 25、集群

- 横向扩展：一般采用新 增节点的方式，增加服务节点的规模来解决性能问题，比如，当一台机器不够用时，可以再增加一台机 器或几台机器来分担流量和业务。

- 纵向扩展：一般采用升级服 务器硬件，增加资源供给，修改服务配置项等方式来解决性能问题，

  ###### 1、全新一主一从实现（修改配置文件容易报错）
  
  ****![image-20240731201828254](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240731201828254.png)

```sql
#主节点配置   10.0.0.171

#创建文件夹
[root@10 ~]$ mkdir -pv /data/mysql/logbin
mkdir: created directory '/data'
mkdir: created directory '/data/mysql'
mkdir: created directory '/data/mysql/logbin'
#给文件夹加权限
[root@10 ~]$ chown -R mysql.mysql /data/mysql/
#修改
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysql/mysqld.log
pid-file=/run/mysqld/mysqld.pid
#指定server-id和存放二进制文件的路径
server-id=171
log_bin=/data/mysql/logbin/mysql-bin
#创建账号默认权限为mysql_native_password 
default_authentication_plugin=mysql_native_password 

#重启服务
#重启服务失败时可先输入命令setenforce 0
#关闭防火墙 (关完systemctl status firewalld.service确认一下)
#重连mysql
#(如果还不行)然后更改文件/etc/selinux/config中将SELINUX的值从enforcing或permissive更改为disabled，然后重启系统。
[root@10 ~]$ systemctl restart mysqld
#查看二进制日志ll /data/mysql/logbin/ #或 mysql> show master logs;
 mysql>  show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       180 | No        |
| mysql-bin.000002 |       157 | No        |
+------------------+-----------+-----------+

#创建账号，给账号授权（账号权限mysql_native_password）
#查看账号权限（select user,host,plugin from mysql.user）
mysql> create user root@'10.0.0.%' identified by '123456';
Query OK, 0 rows affected (0.11 sec)
mysql> grant replication slave on *.* to root@'10.0.0.%';
Query OK, 0 rows affected (0.00 sec)
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

```

```sql
#从节点配置 10.0.0.157
#创建存储二进制文件文件夹
[root@10 ~]$ mkdir -pv /data/mysql/logbin
mkdir: created directory '/data'
mkdir: created directory '/data/mysql'
mkdir: created directory '/data/mysql/logbin'
#授权
[root@10 ~]$ chown -R mysql.mysql /data/mysql/
#修改配置文件
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf

[mysqld]
server-id=157 #指定server-id
read-only #只读模式
log-bin=/data/mysql/logbin/mysql-bin #指定二进制文件路径
#关闭防火墙，临时关闭安全组件，重启系统
[root@10 ~]$ systemctl stop firewalld.service 
[root@10 ~]$ setenforce 0;
#启动服务
[root@10 ~]$ systemctl restart mysqld.service 

#配置主从同步（易出错，核对值）
CHANGE MASTER TO MASTER_HOST='10.0.0.171', MASTER_USER='root', MASTER_PASSWORD='123456',MASTER_PORT=3306,MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=157;
#启动同步
mysql> start slave;
#master节点创建数据库，表，slave节点查看
#查看master/slave节点线程
mysql> show processlist\G
#------------------------------------------------------------------------------#
#实验中出现错误
#1、未关闭安全模块，关闭防火墙
#2、同步不上主文件修改创建用户的加密方式，在主设备配置文件一定要加
default_authentication_plugin=mysql_native_password
#加上之后创建账号，查看账号权限（select user,host,plugin from mysql.user）是不是 mysql_native_password
#3、防火墙未关，用systemctl disable firewalld.service，没有关掉，重新用的systemctl stop firewalld.service
```

**现有数据一主一从实现**

![image-20240731201837949](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240731201837949.png)

```sql
#主节点10.0.0.157
#与全新配置一主一从实现的主设备配置相同
#注意：关闭安全组件 setenforce 0 关闭防火墙 systemctl stop firewalld.service
#二进制文件仍存储在 /data/mysql/logbin/中，并chown -R mysql.mysql /data/mysql/logbin/ 更改权限

#现有主节点日志
mysql> show master logs;
#重置二进制日志
mysql> reset master
mysql> show master logs;

+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       157 | No       |
+------------------+-----------+-----------+
#备份主节点数据
[root@10 ~]$ mysqldump -A -F --source-data=1 --single-transaction >all.sql
#日志被刷新
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+--------- 
| mysql-bin.000001 |       204 | No       |
| mysql-bin.000002 |       157 | No       |
+------------------+-----------+-----------+
2 rows in set (0.00 sec)

#创建账号并授权（注意创建完后 select user,host,plugin from mysql.user查看加密方式 ）
mysql> create user repluser@'10.0.0.%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
Query OK, 0 rows affected (0.01 sec)

#将备份文件复制到10.0.0.153（从设备）
[root@10 ~]$ scp all.sql 10.0.0.153:


#从节点 10.0.0.153
#修改备份文件
[root@10 ~]$ vim all.sql
#找到（可在vim中esc键，然后 :/MASTER_LOG_FILE 搜索MASTER_LOG_FILE）
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=157; 
#加上
CHANGE MASTER TO 
 MASTER_HOST='10.0.0.157',
 MASTER_USER='repluser',
 MASTER_PASSWORD='123456',
 MASTER_PORT=3306,
 MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=157;
#启动服务
[root@10 ~]$ setenforce 0   #关闭安全组件
[root@10 ~]$ systemctl stop firewalld.service #关闭防火墙
[root@10 ~]$ systemctl start mysqld.service   #启动服务
#导入sql文件
mysql> set sql_log_bin=0;
mysql> source /root/all.sql
#查看主从状态
mysql> show slave status\G
#启动同步线程
mysql> start slave;
#查看主从状态
mysql> show slave status\G
#Slave_IO_Running: Yes 
#Slave_SQL_Running: Yes
#线程已启动
#恢复二进制日志
mysql> set @@sql_log_bin=1;
#查看从设备数据库内容是否与主文件相同
#主文件增删改查，查看从设备是否同步
```

**一主多从实现**

![image-20240731201800031](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240731201800031.png)

```sql
#原有主从不变
#新加一台设备，重新配置从节点（与以前配置从节点相同）
```

**级联复制实现**![image-20240731170323339](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240731170323339.png)



```sql
#master主节点配置与之前相同
#slave中间节点配置与之前大部分相同
#更改slave中间节点配置文件（新加一条 log_slave_updates）
[mysqld]
......

server-id=154
read-only
log_slave_updates  #将主服务器的更改记录的自己的二进制文件中，供下一级slave使用
log-bin=/data/mysql/logbin/mysql-bin
#重启mysql
#查看中间节点的同步状态，slave的io线程和sql线程运行
mysql> show slave status\G
 #Slave_IO_Running: Yes
 #Slave_SQL_Running: Yes
#导出中间节点的备份（不用重置二进制文件！！！）
[root@10 ~]$ mysqldump -A -F --single-transaction --source-data=1 > middleall.sql
#将middleall.sql文件复制到最后的从设备（10.0.0.174）上
[root@10 ~]$ scp middle-all.sql 10.0.0.174:

```

```bash
#最后从节点配置（10.0.0.174）
#下载mysql，创建二进制日志存放文件夹/data/mysql/logbin，授权chown -R mysql.mysql /data/mysql/
#删除mysql文件夹内容
[root@10 ~]$ rm -rf /var/lib/mysql/*
#更改配置文件
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf 
[mysqld]
.......

server-id=174
read-only
log-bin=/data/mysql/logbin/mysql-bin
#修改备份文件
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf
HANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=157;
#修改为
CHANGE MASTER TO 
 MASTER_HOST='10.0.0.153', #中间节点的ip
 MASTER_USER='repluser',
 MASTER_PASSWORD='123456',
 MASTER_PORT=3306,
 MASTER_LOG_FILE='mysql-bin.000004', 
 MASTER_LOG_POS=157;
#启动mysql
#临时关闭二进制日志
mysql> set sql_log_bin=0;
#导入备份数据
mysql> source /root/middleall.sql
#开启二进制日志
mysql> set sql_log_bin=1;
#启动同步
mysql> start slave;
#查看主从状态（两个都为yes就可，否则检测用户状态和防火墙）
mysql> show slave status\G
 #Slave_IO_Running: Yes
 #Slave_SQL_Running: Yes
#进行同步测试，在主节点增删改查数据库和表，查看中间节点和最后从节点是否同步
```

主主复制实现（两个设备互为主备）

![image-20240731213247002](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240731213247002.png)

```sql
#根据一主一从模型中，10.0.0.157为主10.0.0.153为从
#只需要把10.0.0.157配置为10.0.0.153的从节点就可

#在原从节点10.0.0.153上
#更改配置文件
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf 
[mysqld]
......
server-id=154
log_slave_updates
log-bin=/data/mysql/logbin/mysql-bin

#重启mysql，查看153的状态
mysql> show slave status\G
#Slave_IO_Running: Yes
#Slave_SQL_Running: Yes

#查看153的二进制日志文件
mysql> show master status;
+------------------+----------+--------------+------------------
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB 
+------------------+----------+--------------+------------------
| mysql-bin.000005 |      157 |              |                  
+------------------+----------+--------------+------------------
1 row in set (0.00 sec)




#在原主节点157上配置
mysql> CHANGE MASTER TO MASTER_HOST='10.0.0.153', MASTER_USER='repluser',
    -> MASTER_PASSWORD='123456', MASTER_PORT=3306,MASTER_LASTER_LOG_POS=157;
Query OK, 0 rows affected, 9 warnings (0.08 sec)
#开始同步
mysql> start slave;
#再次查看
mysql> show slave status\G
#Slave_IO_Running: Yes
#Slave_SQL_Running: Yes

#更改原从节点的数据，如果原主节点可以同步，则实验成功
```

 **半同步复制实现**

```sql
#master 10.0.0.157  slave1 10.0.0.153   slave2 10.0.0.174

#主节点157 
#下载mysql后先修改配置文件，加default_authentication_plugin=mysql_native_password，更改创建账号的权限

#安装semisync_master.so插件
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
#查看是否安装成功
mysql> show plugins;

#所谓安装，其实在mysql库中的plugin表中写入一条数据
mysql> select * from mysql.plugin;
+----------------------+--------------------+
| name                 | dl                 |
+----------------------+--------------------+
| rpl_semi_sync_master | semisync_master.so |
+----------------------+--------------------+
1 row in set (0.00 sec)

#安装完成后还需要手动开启插件
mysql> select @@rpl_semi_sync_master_enabled;
+--------------------------------+
| @@rpl_semi_sync_master_enabled |
+--------------------------------+
|                              0 |
+--------------------------------+
1 row in set (0.00 sec)
#查看当前使用的二进制文件
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       157 | No        |
+------------------+-----------+-----------+
1 row in set (0.00 sec)
#创建主从复制账号，并授权(创建账号之前应在配置文件写入)
mysql> create user repluser@'10.0.0.%' identified by '123456';
Query OK, 0 rows affected (0.02 sec)
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
Query OK, 0 rows affected (0.00 sec)
#修改配置文件
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf 
[mysqld]
server-id=157
rpl_semi_sync_master_enabled        #rpl_semi_sync_master_enabled=1，打开
log-bin=/data/mysql/logbin/mysql-bin #需要提前创建好文件夹并授权
default_authentication_plugin=mysql_native_password
#重启服务
[root@10 ~]$  setenforce 0  #关闭安全组件
[root@10 ~]$ systemctl stop firewalld.service #关闭防火墙
[root@10 ~]$ systemctl restart mysqld  

#再次查看
mysql> select @@rpl_semi_sync_master_enabled;
+--------------------------------+
| @@rpl_semi_sync_master_enabled |
+--------------------------------+
|                              1 |
+--------------------------------+
1 row in set (0.00 sec)


```

```sql
从节点153、174配置

#下载slave插件
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
#查看是否安装
mysql> show plugins;
#查看插件是否开启（0是未开）
mysql> select @@rpl_semi_sync_slave_enabled;
#修改mysql配置文件
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server-id=153 #两个从节点不能相同
read-only   #只读
rpl_semi_sync_slave_enabled #打开从节点的插件
log-bin=/data/mysql/logbin/mysql-bin #需要创建二进制日志的文件夹并授权

#重启服务（关闭安全组件，防火墙）
[root@10 ~]$ systemctl restart mysqld.service

#在slave上开启主从
CHANGE MASTER TO MASTER_HOST='10.0.0.157', MASTER_USER='repluser', MASTER_PASSWORD='123456',MASTER_PORT=3306,MASTER_LOG_FILE='binlog.000001', MASTER_LOG_POS=157;
#开启同步
mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.01 sec)

#查看状态
mysql> show slave status\G
#Slave_IO_Running: Yes
#Slave_SQL_Running: Yes

```

```sql
mysql> show global variables like '%semi%';
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | ON         |#己开启
| rpl_semi_sync_master_timeout              | 10000      |#超时时长,1W毫秒，10S
| rpl_semi_sync_master_trace_level          | 32         |#指定几台slave节点收到binlog后就给客户端返回
| rpl_semi_sync_master_wait_for_slave_count | 1          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |#当前同步策略
+-------------------------------------------+------------+

mysql> show global status like '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 2     |#2个slave节点
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| ......

#主节点创建数据库，表后，自动同步到从数据库，
#半同步状态下，只要有一个slave节点能同步到数据，master 节点就能返回成功。
#两个slave都关闭的情况下 再次执行写操作，等10S后超时，但master节点上写入成功
```

**复制过滤器**

- 只需要在 master 节点上配置一次即可，不需要在 salve 节点上操作；减小了二进制日志中的数 据量，能减少磁盘IO和网络IO。
- 复制过滤可能会出现跨库时同步失败的问题，设置过滤规则一定要考虑业务，**防止连表**，跨库操作失 败，要把业务覆盖全

```sql
#master 节点上配置db1,db2 不写二进制日志

[root@rocky8 ~]# vim /etc/my.cnf
......
[mysqld]
server-id=177
rpl_semi_sync_master_enabled
rpl_semi_sync_master_timeout=3000
log-bin=/data/mysql/logbin/mysql-bin
binlog-ignore-db=db1
binlog-ignore-db=db2

#保存重启后，主节点创建db1，db2，db3数据库并不会同步到从节点
#主节点去掉过滤，删除db1，db2，db3

#slave节点上配置
[root@rocky8 ~]# vim /etc/my.cnf 
....
replicate-do-db=db1
replicate-do-db=db2
replicate-wild-do-table=db%.stu #指定哪些表应该被复制

#重启服务
#重置主从同步
mysql> CHANGE MASTER TO
    ->  MASTER_HOST='10.0.0.157',
    ->  MASTER_USER='repluser',
    ->  MASTER_PASSWORD='123456',
    ->  MASTER_PORT=3306,
    ->  MASTER_LOG_FILE='mysql-bin.000008', MASTER_LOG_POS=157;
Query OK, 0 rows affected, 9 warnings (0.03 sec)
mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.03 sec)

#主节点创建db1，db2，testdb只同步了db1，db2
#主节点db1，db2，创建tab1，tab2，stu表只同步stu表

```

 **GTID 复制**

- GTID 复制不像传统的复制方式（异步复制、半同步复制）需要找到 binlog 文件名和 POS 点，只 需知道 master 节点的 IP、端口、账号、密码即可

- 开启 GTID 后，执行 change master to  master_auto_postion=1 即可，它会自动寻找到相应的位置开始同步。

- 优点：

  便于主从复制的搭建

  GTID 可以知道事务在最开始是在哪个实例上提交的，保证事务全局统一

  截取日志更加方便。跨多文件，判断起点终点更加方便 

  传输日志，可以并发传输。SQL 回放可以更高并发 

  判断主从工作状态更加方便

```sql
#首先创建好主从（主10.0.0.157，从10.0.0.153）

#从节点上还原状态
mysql> stop slave;
mysql> reset slave all;
#删除数据库（系统自带的不删）
mysql> drop database ......;
#修改配置
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf
server-id=153
read-only
log-bin=/data/mysql/logbin/mysql-bin
gtid_mode=ON
enforce_gtid_consistency=ON
#重启服务
```

```sql
#主节点
#删除数据库
#修改配置
[root@10 ~]$ vim /etc/my.cnf.d/mysql-server.cnf
.......
server-id=157
log_bin=/data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
gtid_mode=ON
enforce_gtid_consistency=ON
#重启服务

```

```sql
#从节点设置主从同步
mysql> CHANGE MASTER TO
    ->  MASTER_HOST='10.0.0.157',
    ->  MASTER_USER='repluser',
    ->  MASTER_PASSWORD='123456',
    ->  MASTER_PORT=3306,
    ->  MASTER_AUTO_POSITION=1;
Query OK, 0 rows affected, 8 warnings (0.03 sec)
mysql> start slave;
#在主节点写入数据库表格，从节点测试
```





- **数据不一致如何修复**

重置主从关系，从新复制

- **主从复制出现延迟**

- 升级到 MySQL5.7 以上版本(5.7之前的版本，没有开 GTID 之前，主库可以并发事务，但是 dump 传输时是串行)利用 GTID( MySQL5.6需要手动开启，MySQL5.7 以上默认开启)支持并发传输  
- binlog 及并行多个 SQL 线程。 
- 减少大事务，将大事务拆分成小事务 
- 减少锁
- sync_binlog=1 加快 binlog 更新时间，从而加快日志复制 
- 需要额外的监控工具的辅助 
- 多线程复制：对多个数据库复制 
- 一从多主：Mariadb10 版后支持

**如何避免主从不一致**

主库 binlog 采用 ROW 格式 

主从实例数据库版本保持一致 

主库做好账号权限把控，不可以执行 set sql_log_bin=0

从库开启只读，不允许人为写入 

定期进行主从一致性检验
