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

  

  

  - `mysqladmin -uroot -p123456 -hlocalhost password 13579`修改密码

  ##### 

  ```bash
  
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
4. 