#### linux基础

- 查看当前的终端设备：`root@ubuntu2204:~# tty`
- shell：提供了用户与内核进行交互操作的一种接口，接收用户输入的命令并 把它送入内核去执行
- 显示当前使用的 shell ：`root@ubuntu2204:~# echo ${SHELL}`

- 显示当前系统使用的所有shell：`root@ubuntu2204:~# cat /etc/shells`

```bash
#设置主机名
#临时生效
hostname NAME
#持久生效,支持CentOS7和Ubuntu18.04以上版本
hostnamectl set-hostname NAME
#主机名不支持使用下划线，但支持横线，可使用字母，横线或数字组合
```

```bash
#提示符
echo $PS1 #查看当前命令提示符
[\u@\h \W]\$

#PS1变量中的常用选项

\d #曰期，格式为"星期 月 日"
\H #完整的主机名。如默认主机名"localhost.localdomain"。
\h #写的主机名。如默认主机名"localhost"。
\t #24小时制时间，格式为"HH:MM:SS"。
\T #12小时制时间，格式为"HH:MM:SS"
\A #24小时制时间，格式为"HH:MM"。
\@ #12小时制时间，格式为"HH:MM am/pm"。
\u #当前用户名。
\v #Bash版本信息。
\w #当前所在目录的完整名称。
\W #当前所在目录的最后一个目录。
\# #执行的第几条命令。
\$ #提示符。如果是 root 用户，则会显示提示符为"#"；如果是普通用户，则会显示提示符为"$"
\033[     #开始位   \e[
\033[0m   #结束位   \e[0m
#颜色 30-37  \e[33;40;1mabc\e[0m 
#背景色 40-47
#特殊效果

0 #关闭效果
1 #高亮显示
3 #斜体
4 #下划线
5 #闪烁,闪烁效果与远程工具所在的环境有关
7 #反白显示
8 #隐藏
9 #删除线

#rocky 永久保存echo "PS1='\e[31;1m[\u@\h \W]\\$ \e[0m'" > /etc/profile.d/env.sh
#ubuntu       echo "PS1='\e[31;1m[\u@\h:\w]\$\e[0m '" >> .bashrc
```

```bash
#查看是内部还是外部命令
root@ubuntu2204:~# type echo
echo is a shell builtin

root@ubuntu2204:~# type -a echo
echo is a shell builtin
echo is /usr/bin/echo
echo is /bin/echo

#外部命令搜索路径
#root用户
[root@rokcy8 ~]# echo $PATH
#普通用户
[root@rokcy8 ~]# su - mage
[mage@rokcy8 ~]$ echo $PATH
```

```bash
#当外部命令执行时，默认会从PATH路径下寻找该命令，找到后会将这条命令的路径记录到hash表中，当再次使用该命令时，shell解释器首先会查看hash表，存在将执行之，如果不存在，将会去PATH路径下寻找，
hash   #显示当前终端进程中的hash 缓存
hash -l #显示详细创建此条hash 的命令，可作为输入使用 
hash -p path name #手动创建hash
hash -t name #输出路径
hash -d name #删除指定hash
hash -r #清空所有hash
```

```bash
#别名
alias #显示当前shell进程所有可用的命令别名
alias name #查看指定别名
alias NAME='VALUE' #定义别名NAME，其相当于执行命令VALUE
unalias #撤消别名
unalias -a#撤消所有别名

命令执行优先级
别名 -----> 内部命令 ------>hash--->外部命令
```

##### 查看设备信息

```bash
#lscpu 命令可以查看cpu信息 
#cat /proc/cpuinfo也可看查看到
#查看当前的终端设备：`root@ubuntu2204:~# tty`
#查看内存
[root@ubuntu2204 ~]# free
[root@ubuntu2204 ~]# cat /proc/meminfo
#查看硬盘和分区情况
[root@centos8 ~]# lsblk
[root@centos8 ~]# cat /proc/partitions 
#查看系统架构 arch
#查看内核版本 uname -r

#CentOS8 查看发行版本
[root@centos8 ~]#cat /etc/redhat-release
[root@centos8 ~]#cat /etc/os-release
[root@centos8 ~]#lsb_release -a
#ubuntu
[root@ubuntu2204 ~]# cat /etc/os-release
```

##### 时间

```bash
#date
#显示时区 date -R
#时间戳 date +%s


#对钟clock -s

#设置时区
[root@ubuntu2204 ~]# timedatectl list-timezones

[root@ubuntu2204 ~]# timedatectl set-timezone Asia/Shanghai

[root@ubuntu2204 ~]# timedatectl status

               Local time: Mon 2023-05-08 10:33:48 CST
           Universal time: Mon 2023-05-08 02:33:48 UTC
                 RTC time: Mon 2023-05-08 02:33:48
               Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes

             NTP service: active
         RTC in local TZ: no
         
[root@ubuntu2204 ~]# ll /etc/localtime

lrwxrwxrwx 1 root root 33 May  8 10:33 /etc/localtime -> 
/usr/share/zoneinfo/Asia/Shanghai

#日历
[root@ubuntu2204 ~]# cal 9 1752
```

##### 关机重启

```
shutdown #一分钟后关机
shutdown +10 #10分钟后关机
shutdown 01:02 #1点过两分关机
shutdown -r|--reboot #一分钟后重启
shutdown -r now #现在重启
tdown -H|--halt #一分钟后调用halt 关机
shutdown -P|--poweroff #一分钟后调用poweroff 关机
shutdown -c #取消关机计划

#-r 表示一分钟后重启，如果想立即执行 shutdown -r now
[root@ubuntu2204 ~]# shutdown -r

#取消重启
[root@ubuntu2204 ~]# shutdown -c
```

##### 用户登录信息

whoami：显示当前登录有效用户  

who：系统当前所有的登录会话  

w：系统当前所有的登录会话及所做的操作

##### Screen

```bash
#CentOS7 安装screen
[root@centos7 ~]# yum -y install screen 
#CentOS8 安装screen
[root@centos8 ~]# dnf -y install epel-release
[root@centos8 ~]# dnf -y install screen
#ubuntu
[root@ubuntu ~]# apt install screen

screen -S [SESSION] #创建新screen会话  
screen -x [SESSION] #加入screen会话
screen -r [SESSION] #恢复某screen会话
screen -ls #显示所有已经打开的screen会话
Ctrl+a,d #剥离当前screen会话
exit #退出并关闭screen会话
```

TMUX 

```
#安装
[root@rocky8 ~]# yum install tmux

[root@ubuntu2204 ~]# apt update
root@ubuntu2204 ~]# apt install tmux
#启动和退出
[root@ubuntu2204 ~]# tmux
[root@ubuntu2204 ~]# exit
[exited]
#新建会话
tmux new -s <session-name>
#查看会话
tmux ls
tmux list-session

```

##### 命令行扩展和被括起来的集合

变量 

双引号，弱引用，可以解析内容 单引号，强引用，原样输出

 

命令

`cmd` 展开

$(cmd) 展开

##### 命令行历史

```
history
-c #清空命令历史
-d offset #删除历史中指定的第offset个命令
n #显示最近的n条历史
-a #追加本次会话新执行的命令历史列表至历史文件
-r #读历史文件附加到历史列表
-w #保存历史列表到指定的历史文件
-n #读历史文件中未读过的行到历史列表
-p #展开历史参数成多行，但不存在历史列表中
-s #展开历史参数成一行，附加在历史列表后

```

