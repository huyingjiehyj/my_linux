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
e
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

**监控 Nginx 服务**

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

##### 3、自定义监控项和模版

```bash
#在 Agent 端添加自定义监控项
[root@ubuntu2 ~]$ zabbix_agencat -A  /etc/zabbix/zabbix_agent2.d/item.conf

ESTABLISHEDUserParameter=line_user_count,who|wc -l #当前主机有多少

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

