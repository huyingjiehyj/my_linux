##### 企业级监控系统 ZABBIX

##### 1、zabbix介绍

- 架构 

  - Server：server 是存储所有配 置、统计和操作数据的中央存储库       核心组件

  - 数据存储：Zabbix 收集的所有配置信息以及数据都存储在数据库中

  - web界面

  - proxy：Zabbix proxy 可以代替 Zabbix server 收集性能和可用性数据，

  - agent：主动监控本地资源和应用程序，并将收集到的数据报告给  Zabbix server

  - 数据流

    

![image-20240806211419938](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240806211419938.png)

**Zabbix Server 启动进程** ps aux命令来查看



##### 2、zabbix部署

###### 安装zabbix-server

```bash
#lsb_release -a  查看ubuntu版本号
#进入  https://www.zabbix.com/cn/download 根据要求选择zabbix版本，等
#根据提示进行下载

[root@ubuntu2 ~]$ wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-6+ubuntu24.04_all.deb
[root@ubuntu2 ~]$ dpkg -i zabbix-release_6.0-6+ubuntu24.04_all.deb

#会生成一个源的配置文件
[root@ubuntu ~]# cat /etc/apt/sources.list.d/zabbix.list 
# Zabbix main repository
deb https://repo.zabbix.com/zabbix/6.0/ubuntu noble main
deb-src https://repo.zabbix.com/zabbix/6.0/ubuntu noble main

[root@ubuntu2 ~]$ apt update
#安装Zabbix server，Web前端，agent
[root@ubuntu2 ~]$ apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent

#安装mysql-server (网站没有这一步)
[root@ubuntu2 ~]$ apt install mysql-server
#创建数据库，创建用户，授权
[root@ubuntu2 ~]$ mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by '123456';  #如果在远程主机商创建把localhost改了
mysql> grant all privileges on zabbix.* to zabbix@localhost;   #授权
mysql> set global log_bin_trust_function_creators = 1;#保证在不开启二进制日志的情况下创建函数
mysql> quit;

#导入数据表(等待初始化)
[root@ubuntu2 ~]$ dpkg -L zabbix-sql-scripts
[root@ubuntu2 ~]$ zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p123456 zabbix

#改回配置
[root@ubuntu2 ~]$ mysql
mysql> set global log_bin_trust_function_creators = 0;

#更改zabbix配置文件，（如果在远程主机上创建的数据库，需要改DBhost）
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_server.conf
...
DBPassword=123456
...
#修改zabbix中nginx配置
[root@ubuntu2 ~]$ vim /etc/zabbix/nginx.conf
server {
       listen          80;
       server_name     zabbix.m99-magedu.com;
#重启服务
[root@ubuntu2 ~]$ systemctl restart zabbix-server zabbix-agent nginx php8.3-fpm
[root@ubuntu2 ~]$ systemctl enable zabbix-server zabbix-agent nginx php8.3-fpm


#安装完成后，生成的PHP配置文件，后续数据库配置如果发生了变化，可以直接修改此文件
[root@ubuntu2 ~]$ ll /usr/share/zabbix/conf/zabbix.conf.php
lrwxrwxrwx 1 root root 31 Jul 15 08:27 /usr/share/zabbix/conf/zabbix.conf.php -> /etc/zabbix/web/zabbix.conf.php

#window写入host 10.0.0.181 zabbix.m99-magedu.com
#在浏览器中访问 zabbix.m99-magedu.com，根据提示点击下一步，完成 web 的安装，默认登录用户名和密码是 Admin, zabbix

#安装中文语言包，重启服务
[root@ubuntu2 ~]$ apt install language-pack-zh-hans -y
[root@ubuntu2 ~]$ systemctl restart zabbix-server zabbix-agent nginx php8.3-fpm
##zabbix 配置中文语言

#点击导航栏中的 User setting，Profile，在语言栏选择简体中文

#在 监测，主机 菜单中，点击 图形，会看到有乱码

#在 windows 的 C:\Windows\Fonts 目录中，找到自己喜欢的字体文件，上传为 
/usr/share/zabbix/assets/fonts/graphfont.ttf
#再刷新页面即可
[root@ubuntu ~]# ls /usr/share/zabbix/assets/fonts/
graphfont.ttf graphfont.ttf.bak

```

###### 安装 Zabbix Agent

```bash
#LINUX 安装agent
# 在https://www.zabbix.com/cn/download 选择被监控主机的型号，
#ZABBIX COMPONENT 选择 Agent 2
#根据提示下载
[root@ubuntu2 ~]$ wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-6+ubuntu24.04_all.deb
[root@ubuntu2 ~]$ dpkg -i zabbix-release_6.0-6+ubuntu24.04_all.deb
[root@ubuntu2 ~]$ apt update
[root@ubuntu2 ~]$ apt install zabbix-agent2 zabbix-agent2-plugin-*
#查看端口（10500）和服务
[root@ubuntu2 ~]$ systemctl status zabbix-agent2.service
root@ubuntu2 ~]$ ss -tnlp | grep zabbix



#windows 安装Agent
#在 https://www.zabbix.com/cn/download_agents 下载
#windows 客户端默认安装路径是 C:\Program Files\Zabbix Agent 2\
#配置文件是C:\Program Files\Zabbix Agent 2\zabbix_agent2.conf
#命令窗口 netstat -na | findstr 10050 查看是否开启端口

```

配置zabbix Agent

```bash
#在 server 上安装 zabbix-get
[root@ubuntu2 ~]$ apt install zabbix-get -y
#测试连接agent 如果报错连接失败，如果为1，连接成功
[root@ubuntu2 ~]$ zabbix_get -s 10.0.0.161 -k "agent.ping"
#报错

#agent上更改配置
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_agent2.conf
Server=127.0.0.1,10.0.0.0/24 #所有10网段可连
#重启服务
[root@ubuntu2 ~]$ systemctl restart zabbix-agent2.service 

#server上测试
[root@ubuntu2 ~]$ zabbix_get -s 10.0.0.161 -k "agent.ping"
1
[root@ubuntu2 ~]$ echo $?
0

#web页面添加agent主机

#linux
# 监测，主机，创建主机
# 主机名称：10.0.0.161
# 可见名称：m99-ubuntu-217
# 模板：选择 Linux by Zabbix agent，可以选多个，每个模板代表不同的监控项
# 群组，可以选择，也可以创建新的
# interfaces: 点击添加，选择 客户端，填 10.0.0.161
# 点击 添加，该主机就会出现在主机列表中，等待一会儿刷新页面，就能看到远程主机的数据

#windows
# 在 Server 节点的 监测，主机 页面选择 创建主机
# 模板选择 Windows by Zabbix agent
# Interfaces 选择客户端，填写 windows 主机 IP 10.0.0.1
# 其它自定义
# 点击 添加
```

##### 3、**监控 Nginx 服务**

-  **Nginx by Zabbix Agent 监控 Nginx**
- 通过 Zabbix Agent 传输 Nginx 相关数据

```bash
#在己安装 Zabbix Agent 的主机上安装 Nginx
[root@ubuntu2 ~]$ apt install nginx
#配置状态页
[root@ubuntu2 ~]$ vim /etc/nginx/sites-enabled/default 
#配置状态页

[root@ubuntu ~]# cat /etc/nginx/sites-enabled/default
...
 location / {
 try_files $uri $uri/ =404;
 }
 #添加
 location = /ngx_status { # 默认是写 basic_status 
   stub_status;
 }
...
#重启，在浏览器测试10.0.0.161/ngx_status
[root@ubuntu2 ~]$ systemctl restart nginx.service 

#在server端 监测 主机页面 找到nginx的主机，点击名称，选择配置，在在模板选项增加 Nginx by Zabbix Agent 模板
# 然后切换到 宏 ，继承及主机宏 找到 {$NGINX.STUB_STATUS.PATH}，将值修改为 ngx_status
#更新
#在列表页点击 数据，根据名称搜索 nginx，就能看到 nginx 数据了
```

##### 4、监控PHP

- **使用 Nginx by HTTP 监控 Nginx** 
- 通过 HTTP 协议传输 Nginx 相关数据，无需部署 Zabbix Agent

```bash
#主机10.0.0.163 无zabbix Agent 安装nginx
#更改配置文件
[root@ubuntu2 ~]$ vim /etc/nginx/sites-enabled/default 
...
location / {
 try_files $uri $uri/ =404;
 }
 location = /basic_status {
   stub_status;
 }
...
#重启，访问10.0.0.163/basic_status
[root@ubuntu2 ~]$ systemctl restart nginx.service

#新建主机 模版选择nginx by HTTP
# interface 类型任选，客户端|SNMP|JMX|IPMI 都可以，只需要填对IP地址，无关端口号
# 点击 添加 即可
# 如果需要修改数据刷新频率，可以进入 配置，主机 页面，选择 监控项，选择所有监控项 批量更新 间隔
```

**PHP-FPM by Zabbix agent** 

通过 Zabbix Agent 传输 PHP-FPM 相关数据

```bash
#在己安装 Zabbix Agent，nginx 的主机上安装 php，并开启状态页，配置 nginx 反向代理
[root@ubuntu2 ~]$ apt install php-fpm -y
#修改php和nginx配置文件
[root@ubuntu2 ~]$ vim /etc/php/8.3/fpm/pool.d/www.conf

listen = /run/php/php8.3-fpm.sock
pm.status_path = /php_status

ping.path = /ping
[root@ubuntu2 ~]$ vim /etc/nginx/sites-enabled/default

 location ~ \.php|/php_status|/ping$ {
          fastcgi_pass unix:/run/php/php8.3-fpm.sock;
          include fastcgi_params;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
 }
 
 #重启
 [root@ubuntu2 ~]$ systemctl restart php8.3-fpm.service 
 [root@ubuntu2 ~]$ systemctl restart nginx.service
#测试10.0.0.161/ping 10.0.0.161/php_status

#在server端 web界面 主机，找到之前的 Nginx by Zabbix Agent 的主机
#点击主机名 配置 添加模版 PHP-FPM by Zabbix agent
#修改宏 {$PHP_FPM.STATUS.PAGE} 值为 php_status
#点击添加
#如果需要修改数据刷新频率，可以进入 配置，主机 页面，选择 监控项，选择所有监控项 批量更新 间隔

```

**使用 PHP-FPM by HTTP 监控PHP**

通过 HTTP 协议传输 PHP-FPM 相关数据，无需部署 Zabbix Agent

```bash
#安装php
#修改php，nginx配置文件（与PHP-FPM by Zabbix agent相同）
#重启
#在浏览器测试 10.0.0.163/ping 10.0.0.163/php_status

#监测，主机，点击主机名，配置
#添加模版 {$PHP_FPM.STATUS.PAGE} 值为 php_status,{php_FPM.HOST}值为10.0.0.163
#添加
```

##### 5、自定义监控项和模版

```bash
#在 Agent 端添加自定义监控项
[root@ubuntu2 ~]$ zabbix_agencat -A  /etc/zabbix/zabbix_agent2.d/item.conf

ESTABLISHEDUserParameter=line_user_count,who|wc -l #当前主机有多少用户登录
UserParameter=tcp_listen,netstat -tna | grep -c LISTEN  #有多少listen状态tcp进程
UserParameter=tcp_established,netstat -tna | grep -c ESTABLISHED #有多少ESTABLISHED状态的tcp进程
#重启
[root@ubuntu2 ~]$ systemctl restart zabbix-agent2.service 
#测试
[root@ubuntu2 ~]$ systemctl rzabbix_agent2 -t "line_user_count"
...
line_user_count                               [s|2]


#server端测试
[root@ubuntu2 ~]$ zabbix_get -s 10.0.0.162 -k "line_user_count"
2

#server端 配置，主机，点击对应主机、监控项、创建监控项
#名称：用户登录数
#键值：line_user_count
#更新间隔 10s
#点击测试，如果获取值添加
#监测、主机、对应主机、点击最新数据、输入名称、应用
```

```bash
#现在agent端添加自定义监控项
#配置 模版 页面 创建模版
#模版名称 M99-temp 写群组 添加

#配置 模版 监控项 找到新建的M99-temp 点击监控项 创建监控项
#名称 键值line_user_count 更新时间10s 添加
#关联主机

```

**传参**

```bash
#更改配置文件
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_agent2.d/item.conf 
UserParameter=line_user_count,who|wc -l
UserParameter=tcp_status[*],netstat -tna | grep -c $1
#测试
[root@ubuntu2 ~]$ zabbix_agent2 -t "tcp_status[LISTEN]"
tcp_status[LISTEN]                            [s|8]
[root@ubuntu2 ~]$ zabbix_agent2 -t "tcp_status[ESTABLISHED]"
tcp_status[ESTABLISHED]                       [s|2]
#重启
[root@ubuntu2 ~]$ systemctl restart zabbix-agent2.service 
#服务端修改模版
键值tcp_status[LISTEN]     \    tcp_status[ESTABLISHED] 
```

**监控项值映射**

```bash
#配置模版M99-temp
#创建监控项tcp_80_status 键值 net.tcp.listen[80] 添加
#tcp_80_status，查看该监控项的数据，只能看到 0，1 这种数字
#模版，找到M99-temp 点击 值映射 添加
#名称 tcp_80_status_mapping    1=up   0=down 更新
#配置 模版 监控项 点击tcp_80_status 编辑页面 关联值映射
#回到 监测，主机页面，查看 tcp_80_status 监控数据，就不再是只有 0，1 这种数字了

```

##### 6、配置触发器

```bash
#配置 模版页面 找到M99-temp 点击触发器
#名称 tcp_80_down  严重 
#表达式 添加 监控项tcp_80_status 添加
#配置 模版 可以看到触发器（1）
#关闭对应主机nginx 产生告警 重启nginx 告警消失
#用户设置 配置中  正在发送消息 可打开更改声音
```

##### 7、图表

```bash
#配置 模版 找到M99-temp 点击图形 创建图形
#名称test 监控项选择tcp_80_status 
#添加
#监测 主机 对应主机 图形
```

##### 8、自定义表盘

配置、模版、找到模版、仪表盘、创建仪表盘

##### 9、设置报警: 管理 媒介



##### 10、故障自愈

```bash
#Agent主机
[root@ubuntu ~]# cat /etc/zabbix/zabbix_agent2.conf
AllowKey=system.run[*] #可以执行Server 端发来的任何命令
UnsafeUserParameters=1 #可以支持特殊字符，转义字符
#重启服务
[root@ubuntu ~]# systemctl restart zabbix-agent2.service 

#添加 sudo 授权
[root@ubuntu ~]# cat /etc/sudoers | grep zabbix #只读是visuo编写 ctrl+0然后enter，然后ctrl+x退出
...
User zabbix may run the following commands on ubuntu:
   (ALL : ALL) NOPASSWD: ALL
   
   
# 管理，脚本，创建脚本
## 名称：nginx restart
## Scope：动作操作
## 类型：脚本
## 执行在：Zabbix客户端
## 命令：sudo systemctl restart nginx
## 添加

# 配置，动作，Trigger actions，创建动作
## 名称：nginx故障自愈
## 条件：添加
## 类型：触发器
## 操作者：等于
## 触发器：tcp_80_down
## 添加

# 操作
## 操作，添加
## Operation：nginx restart
## 目标列表：当前主机
## 保存
# 添加
#测试是否能实现服务自愈
```

##### 11、主动模式和被动模式

被动模式：是指 Server 端主动向 Agent 端发起获取数据的请求，每次获取一个监控项的数据

主动模式：是指 Agent 端主动向 Server 端上报数据，每次上报的数据包含所有监控项的内容

```bash
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_agent2.conf
Server=127.0.0.1,10.0.0.0/24 #表示可以来连接当前 Agent 的 Server 端IP

#被动模式下，server端能看到大量连接
[root@ubuntu2 ~]$ ss -tan | grep 10050

#修改 Agent 端配置为主动模式，让 Agent 批量上报数据
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_agent2.conf 
ServerActive=10.0.0.181 #此处修改为 Zabbix Server 地址
Hostname=Zabbix server #在主动模式下此项的值要与 Server 端的主机名称保持一致
#重启
[root@ubuntu2 ~]$ systemctl restart zabbix-agent2.service

```

##### 12、监控 Java 程序

Zabbix 不支持直接监控 JAVA 应用，需要使用 Java Gateway 做代理才能从 JAVA 应用程序中获取数据（java Gateway 10052端口）

JAVA 应用要求开启 JMX才被被监控



```bash
#监控主机上安装tomcat
[root@ubuntu ~]# apt update;apt install tomcat10 -y
#查看页面10.0.0.162:8080
#开启tomcat JMX远程连接功能
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_agent2.conf 
ATALINA_OPTS="$CATALINA__OPTS -Dcom.sun.management.jmxremote -
Dcom.sun.management.jmxremote.port=12345 -
Dcom.sun.management.jmxremote.authenticate=false -
Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=10.0.0.162"

# -Dcom.sun.management.jmxremote #开启远程JMX连接
# -Dcom.sun.management.jmxremote.port=12345 #JMX端口
# -Dcom.sun.management.jmxremote.authenticate=false #JMX连接不用认证
# -Dcom.sun.management.jmxremote.ssl=false #JMX连接不使用SSL认证
# -Djava.rmi.server.hostname=10.0.0.220 #要被监控的tomcat的主机IP
[root@ubuntu2 ~]$ systemctl restart zabbix-agent2.service

```

```bash
#监控主机server上
#在Zabbix Server 端同一台机上安装 javaGateway，可以分开不同主机
[root@ubuntu ~]# apt install zabbix-java-gateway -y

#默认监听10052端口
[root@ubuntu ~]# ss -tnlp | grep java

#修改 Zabbix Server 配置
[root@ubuntu ~]# cat /etc/zabbix/zabbix_server.conf
JavaGateway=127.0.0.1 #JMX 地址
StartJavaPollers=2 #Server 端开启进程连接JMX

#重启服务，能看到两个zabbix进程
[root@ubuntu ~]# systemctl restart zabbix-server.service 
[root@ubuntu ~]# ps aux | grep "java poller" #两个zabbix进程

#sever端，添加主机，模版选Generic Java JMX
#查看最新数据
```

##### 13、SNMP 基础

snmp：简单网络管理协议，属于TCP/IP五层协议中的应用 层协议，用于网络管理的协议，SNMP主要用于网络设备的管理

SNMP网络设备分为NMS和Agent两种：

- nms：是snmp网络管理者
- agent： 是snmp网络被管理者
- NMS和Agent之间通过SNMP协议来交互管理信息,在SNMP中，用得最多的协议还是UD

MIB：管理信息库

- 公有MIB
- 私有MIB

OID：MIB存放的每个对象都有唯一的对象标识OID，OID是一种数据类型

OID 是一种数据类型，它指明一种“授权”命名的 对象，“授权” 的意思就是这些标识不是随便分配的，它是由一些权威机构进行管理和分配的，对象标识 是一个整数序列，以点（“.”）分隔，这些整数构成一个树型结构，类似于DNS或Unix的文件系统。对象 标识从树的顶部开始，顶部没有标识，以root表示

```bash
#安装snmp软件模拟网络设备
#在被监控端
[root@ubuntu2 ~]$ apt update;apt install snmpd -y
#只监听了161
[root@ubuntu2 ~]$ ss -unlp | grep snmpd
#修改配置文件
agentaddress  0.0.0.0                         #监听所有IP的161

view   systemonly included   .1.3.6.1.2.1.1
view   systemonly included   .1.3.6.1.2.1.25.1
view   systemonly included   .1                #添加此行，可以获取所有以 .1 

rocommunity public default -V systemonly       #默认group 值是 public开头的指标
#重启，查看
[root@ubuntu2 ~]$ systemctl restart snmpd
[root@ubuntu2 ~]$ ss -unlp | grep snmpd

```

```bash
#监控端主机
#测试远程snmp协议抓取数据

[root@ubuntu2 ~]$ apt install snmp -y
#测试
[root@ubuntu2 ~]$ snmpwalk -v 2c -cpublic 10.0.0.161 .1.3.6.1.2.1.1.5.0
iso.3.6.1.2.1.1.5.0 = STRING: "ubuntu2"
[root@ubuntu2 ~]$ snmpwalk -v 2c -cpublic 10.0.0.161 .1.3.6.1.2.1.1.1
iso.3.6.1.2.1.1.1.0 = STRING: "Linux ubuntu2 6.8.0-39-generic #39-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul  5 21:49:14 UTC 2024 x86_64"

```

```bash
#创建主机 模块Linux by SNMP 
#SNMP 10.0.0.220 161
# SNMP version：SNMPv2
# SNMP community：{$SNMP_COMMUNITY}
#添加
```

##### 14、  Zabbix 分布式监控

Zabbix Proxy 是一个数据收集器,只执行数据收集

不计算触发器、不处理事件、不发送报警，无Web管理界面



Zabbix Proxy 也分主动模式和被动模式，通信方式与 Zabbix Server 主动模式和被动模式一样

- 主动模式是zabbix-proxy主动向zabbix-server周期上报获取的zabbix agent数据
- 被动模式是等待 Zabbix Server的连接，并接受 Zabbix Server 发送的监控项 指令，然后再由 Zabbix Proxy 向Zabbix Agent 发起请求获取数据

15、Zabbix Proxy 实现分布式监控

![image-20240809193902312](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240809193902312.png)

**主动模式**

```bash
#Zabbix Proxy 10.0.0.217
#修改主机名
[root@ubuntu ~]# hostnamectl hostname proxy-active
#配置zabbix 源
[root@ubuntu ~]# wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-6+ubuntu24.04_all.deb
[root@ubuntu ~]# dpkg -i zabbix-release_6.0-6+ubuntu24.04_all.deb
[root@ubuntu ~]# apt update
[root@ubuntu ~]# apt install zabbix-proxy-mysql zabbix-sql-scripts
#下载mysql数据库
#配置账号并授权
mysql> create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by '123456';
mysql> grant all privileges on zabbix_proxy.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
#导入数据
[root@ubuntu ~]# cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -p '123456'
#还原mysql数据
mysql> set global log_bin_trust_function_creators = 0;
#修改配置
[root@ubuntu ~]# vim /etc/zabbix/zabbix_proxy.conf
Server=10.0.0.216  #zabbix server服务器地址

Hostname=proxy-active #当前节点名称

# DBHost=localhost
DBName=zabbix_proxy  #数据库名称
DBUser=zabbix
DBPassword=123456

#加开机启动
[root@proxy-active ~]# systemctl enable --now zabbix-proxy.service
#监听端口10051
```

```bash
#主动模式Agent配置
#修改主机名
[root@ubuntu ~]# hostnamectl hostname agent-active
#下载部署Agent
# wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-6+ubuntu24.04_all.deb
# dpkg -i zabbix-release_6.0-6+ubuntu24.04_all.deb
# apt update
#apt install zabbix-agent

#修改配置文件，agent到proxy是主动模式
[root@agent-active ~]# vim /etc/zabbix/zabbix_agentd.conf 
ServerActive=10.0.0.217 #主动proxy的地址
Hostname=agent-active #主机名
#重启服务
[root@agent-active ~]# systemctl restart zabbix-agent.service
```

```bash
#Zabbix Server 
#管理 proxy 创建proxy 
#名称：proxy-active 主动式 代理地址：10.0.0.217
#配置 主机 创建主机 
#主机名称：agent-active 可见名称 agent-active-217 
#模版：Linux by Zabbix agent active  #群组M99
#接口10.0.0.218 10050
#proxy代理程序监测：proxy-active #更新
#最新数据
```

**被动模式**

```bash
#proxy主机192.168.10.217（仅主机）（10051）
#修改主机名
[root@ubuntu ~]# hostnamectl hostname proxy-paasive
#下载配置proxy、数据库与上面一样
#配置文件
[root@proxy-passive ~]# cat /etc/zabbix/zabbix_proxy.conf
ProxyMode=1                         #被动模式
Server=192.168.10.216               #zabbix server 地址
Hostname=proxy-passive

# DBHost=localhost  #数据库
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=123456

#开机启动
[root@proxy-active ~]# systemctl enable --now zabbix-proxy.service
```

```bash
#zabbix agent 192.168.10.218
#修改主机名
[root@ubuntu ~]# hostnamectl hostname agent-passive
#下载部署agent
#修改配置
[root@agent-passive ~]# cat /etc/zabbix/zabbix_agentd.conf
Server=192.168.10.217
Hostname=agent-passive

#重启服务
[root@agent-passive ~]# systemctl restart zabbix-agent.service

```

```bash
#server端 
#Zabbix Server 
#管理 proxy 创建proxy 
#名称：proxy-passive 主动式 代理地址：192.168.0.217 端口10051
#配置 主机 创建主机 
#主机名称：agent-active 可见名称 agent-passive-218 
#模版：Linux by Zabbix agent #群组M99
#接口192.168.10.218 10050
#proxy代理程序监测：proxy-passive #更新
#最新数据
```

##### 15、Zabbix 实现自动化运维

当大量主机都己经安装了 Zabbix Agent 或 Snmp 后，可以利用自动发现的功能，让 Zabbix Server 自 动扫描预先配置好的 IP 地址网段，实现自动添加主机，自动关联模板等功能

优点：快速的部署 Zabbix、简化 Zabbix 管理，可管理大量主机

**网络发现**

```bash
#客户端下载zabbix-get 被监控主机下载agent，配置Server=127.0.0.1,10.0.0.0/24

#在 Zabbix Server 端测试
[root@ubuntu ~]# zabbix_get -s 10.0.0.217 -k system.uname
Linux agent-217 6.8.0-31-generic #31-Ubuntu SMP PREEMPT_DYNAMIC Sat Apr 20 
00:40:06 UTC 2024 x86_64
[root@ubuntu ~]# zabbix_get -s 10.0.0.218 -k system.uname
Linux agent-218 6.8.0-31-generic #31-Ubuntu SMP PREEMPT_DYNAMIC Sat Apr 20 
00:40:06 UTC 2024 x86_64

Zabbix Server 端，配置，自动发现，创建发现规则

## 名称：M99-discover
## 由agent代理程序自动发现：没有 agent 代理程序

## IP 范围：10.0.0.215-220
## 更新间隔：1m
## 检查：Zabbix 客户端 "system.uname"
## 设备唯一性准则：IP地址

## 主机名称：IP地址
## 可见的名称：主机名称
## 己启用：勾选
# 保存

# Zabbix Server 端，配置，动作，发现动作，创建动作

## 动作

## 名称：auto-discovery-action
## 计算方式：与       A and B and C
## 条件： A 接收到的值 包含 Linux 
## B 自动发现状态 等于 上 
## C   服务类型 等于 Zabbix 客户端 
## 已启用：勾选
## 操作
## 添加到主机群组: M99
## 链接到模板: Linux by Zabbix agent
# 添加

# Zabbix Server 端，配置，动作，发现动作，创建动作

## 动作

## 名称：auto-discovery-remove
## 计算方式：与       A and B and C
## 条件： A 在线/不在线 大于等于 10 
## B 自动发现状态 等于 下
## C   服务类型 等于 Zabbix 客户端 
## 已启用：勾选
## 操作
## 发送消息给用户: Admin (Zabbix Administrator) 通过 email-notify
## 移除主机
# 添加

#刷新页面

```

**自动注册**

```bash
#两个被监控主机下载zabbix-agent
#配置
[root@ubuntu2 ~]$ vim /etc/zabbix/zabbix_agentd.conf
Server=127.0.0.1
ServerActive=10.0.0.216
Hostname=agent-1
HostMetadata=group1 #用于zabbix server端验证 
[root@ubuntu2 ~]$ systemctl restart zabbix-agent.service 

#zabbix server页面
#配置 动作 自动注册动作 创建动作
#名称 auto-register  条件 添加 主机元数据 匹配 值group1
#操作 添加主机群组M99 连接到模版 Linux by Zabbix agent active
#更新
#刷新页面，重启agent 发现多出自动注册的主机
```

##### 16、Zabbix API 实现自动化运维

API：应用程序接口

zabbix优化方法：

- 数据库：写多读少，
- 采用主动模式
- 使用zabbix proxy代理模式监控远程主机
- 使用自定义模版，删除无用监控项
- 减少历史数据保存周期，适当增加取值周期
- 调节缓存和进程、
- 

##### 17、zabbix高可用

