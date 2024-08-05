# lvs

##### 1、扩展

- 横向扩展： 加设备
- 纵向扩展：垂直扩展，升级设备提升性能

在大多数情况下，我们会采用水平扩展的方式来扩展系统性能，

##### 2、集群

- 负载均衡集群 （LB集群）：

- 高可用集群（HA集群）：

- 高性能计算集群（HPC集群）：

- **集群分布式区别**

  集群是同一个业务系统部署在多台服务器上，集群中每一台服务器实现的功能没有差别，数据和代码都 是一样的，

  分布式是一个业务系统被拆分成多个子业务，部署在多台服务器上，每台服务器实现的功能是有差别的，数据和代码也是不一样的，分布式系统中每台服务器完成一 个小功能，所有的功能加起来，才是完整的业务

##### 3、调度算法（4静态+6动态）

4静态

- RR 轮询算法，轮流转发，每个服务器请求量相同
- WRR  加权轮询算法  根据权重进行转发，被调度比例=当前rs权重/总权重
- SH 源IP地址hash ：将来自于同一个 IP 地址的客户端请求调度到后端同一台 RS 服务器上，从而实现会话保持
- DH 目标IP地址hash：客户端的请求第一次被调度到某到 RS 服务器后，其 后续的请求都将会被发往同一台 RS 服务器，一般用于正向代理缓存场景

6动态

- Lc 最少连接算法，将前端请求调度到最少后端rs服务器上
- wlc加权最少连接算法  LVS 的默认调度算法 具有较高权重值的 RS 服务器将承受较大比例的活动连接负载
- SED最短延迟调度算法
- NQ最少队列调度算法/永不排队调度算法，初始的时候先做一次轮循，保证每台  RS 都至少被调度一次，后续使用 SED 调度算法
- LBLC基于局部性的最少链接调度算法，本质是动态的 DH 算法
- LBLCR带复制的基于局部性的最少链接调 度算法

##### 3、lvs工作模式（nat模式，DR模式，tun模式）

###### （1）nat模式

​	![image-20240731112245238](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240731112245238.png)

| 报文顺序 | 源                          | 目标                        |
| -------- | --------------------------- | --------------------------- |
| 1        | client：49.89.179.13:12345  | LVS-VIP：219.142.145.239:80 |
| 2        | client：49.89.179.13:12345  | RS-RIP：192.168.10.110:80   |
| 3        | RS-RIP：192.168.10.110:80   | client：49.89.179.13:12345  |
| 4        | LVS-VIP：219.142.145.239:80 | client：49.89.179.13:12345  |

LVS 将请求报文中的目标地址和目标端口修改为后端的 RS 服务器的 RIP 和 PORT 进行转发

**特点：**

- DIP和RIP在同一ip网络，RS网关指向DIP ，应使用私网地址
- 支持端口映射，可修改请求报文的目标 PORT
- 请求，响应报文都由lvs转发
- Lvs必须是linux系统，rs可以是任何系统
- Lvs主机需开启ip_forward转发

###### （2）DR模式（直接路由）

![image-20240802104351044](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802104351044.png)

![image-20240802105220245](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802105220245.png)

###### （3） TUN 模式

![image-20240802105925909](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802105925909.png)

![image-20240802105947432](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802105947432.png)

###### （4） FULL NAT 模式

![image-20240802110150709](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802110150709.png)

![image-20240802110347644](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802110347644.png)







##### 4、lvs比较

![image-20240802110928598](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240802110928598.png)

NAT 与 FULL NAT： 

请求报文和响应报文都经由 LVS，NAT 模式下的 RIP 网关要指向 DIP，FULL NAT 模式下无此要求；

 DR 与 TUN： 

请求报文经由 LVS，响应报文由 RS 自行转发；DR 模式通过修改 MAC 首部实现，通过 MAC 网络转发；

TUN 模式能过在原 IP 报文之外封装新的 IP 首部实现转发，支持跨公网通信







```bash
#ubuntu上更改时区为上海
[root@ubuntu2 ~]$ timedatectl
               Local time: Fri 2024-08-02 07:03:13 UTC
           Universal time: Fri 2024-08-02 07:03:13 UTC
                 RTC time: Fri 2024-08-02 07:03:12
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
[root@ubuntu2 ~]$ timedatectl set-timezone Asia/Shanghai
```





##### 5、Lvs负载均衡实现

前置工作：

- 在实验环境中，有 Client，Router，LVS，RS 这几种角色的服务器，我们统一使用 ubuntu2204 操作系 统的主机

- 所有主机关闭防火墙（如果有红帽系统，关闭selinux）

- 统一网卡名称

  | 角色   | 数量 | 备注                                                         |
  | ------ | ---- | ------------------------------------------------------------ |
  | Client | 1    | 清空防火墙规则                                               |
  | Router | 1    | 清空防火墙规则，安装 net-tools，开启 ip_forward              |
  | Lvs    | 1    | 清空防火墙规则，安装 net-tools，ipvsadm                      |
  | RS     | 2    | 清空防火墙规则，安装 net-tools，安装 apache 监听88，安装 nginx 监听 80， 并设置不同的默认页面 |

  ```bash
  #统一网卡名称
  [root@ubuntu ~]# vim /etc/default/grub
  GRUB_CMDLINE_LINUX=" net.ifnames=0"
  
  #然后修改网卡配置文件，将对应的 ens160，ens233 这种改成 eth0,eth1 等
  [root@ubuntu ~]# cat /etc/netplan/eth0.yaml
  network:
   ethernets:
     eth0:
       addresses: [192.168.10.100/24]
       routes: [{to: default,via: 192.168.10.200}]
   version: 2
   
  #重新生成 grub 文件并重启  
  [root@ubuntu ~]# grub-mkconfig -o /boot/grub/grub.cfg;reboot
  
  #关闭防火墙  
  [root@ubuntu2 ~]$ systemctl disable --now  ufw
  #所有主机清空 iptables 规则
  [root@ubuntu ~]# iptables -F
  
  #下载net-tools工具包
  [root@ubuntu ~]# apt install net-tools -y
  
  #Router 主机开启 ip_forward 
  [root@ubuntu ~]# echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf ^C
  [root@ubuntu ~]# sysctl -p
  net.ipv4.ip_forward = 1
  
  ```

  

###### （1）NAT模式实现

![image-20240803202354608](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240803202354608.png)

- client   eth0 网卡配置 192.168.10.100/24；仅主机模式；网关指向 192.168.10.200;

- Lvs       eth1 网卡配置 VIP 192.168.10.200/24; 仅主机模式 VIP

  ​             eth0 网卡配置 DIP 10.0.0.8/24；NAT 模式 DIP

- RS1     eth0 网卡配置 RIP 10.0.0.110/24； NAT 模式；网关指向 10.0.0.8;   RIP

- RS2      eth0 网卡配置 RIP 10.0.0.120/24；NAT 模式；网关指向 10.0.0.8;   RIP

  

```bash
#client网卡配置

#配置网关，先在vmware上将网络适配器调整到仅主机模式
[root@ubuntu2 ~]$ vim /etc/netplan/50-cloud-init.yaml
network:
 ethernets:
   eth0:
      #dhcp4: true

     addresses: [192.168.10.100/24]
     routes: [{to: default,via: 192.168.10.200}] #网关
 version: 2
#保存网卡配置
[root@ubuntu2 ~]$ netplan apply
```

```bash
#Lvs网卡配置
#eth0
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml
network:
 ethernets:
   eth1:
     addresses: [10.0.0.8/24]
 version: 2
 
[root@ubuntu2 ~]$ netplan apply
#eth1（vmware上添加一块网卡，设置仅主机模式，在/etc/netplan/下创建eth1.yaml配置文件 ）
[root@ubuntu2 ~]$ cat /etc/netplan/eth1.yaml 
network:
 ethernets:
   eth0:
     addresses: [192.168.10.200/24]
 version: 2
 
[root@ubuntu2 ~]$ netplan apply
#开启ip_forward
#下载ipvsadm查看 apt install ipvsadm

```

```bash
#RS-1、RS-2网卡配置 网关指向Lvs上的DIP 同网段的
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml
network:
 ethernets:
   eth0:
     addresses: [10.0.0.110/24] #写RS-1/RS-2 的ip地址
     routes: [{to: default,via: 10.0.0.8}]
 version: 2
 #下载nginx apt install nginx
 #更改nginx页面
[root@ubuntu2 ~]$ vim /var/www/html/index.nginx-debian.html
10.0.0.110 nginx
#另一台写 10.0.0.120 nginx
```

```bash
#Lvs上配置ipvsadm
[root@ubuntu2 ~]$ ipvsadm -A -t 10.0.0.8:80 
[root@ubuntu2 ~]$ ipvsadm -a -t 10.0.0.8:80 -r 10.0.0.110 -m 
[root@ubuntu2 ~]$ ipvsadm -a -t 10.0.0.8:80 -r 10.0.0.120 -m
#查看
[root@ubuntu2 ~]$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.8:80 wlc
  -> 10.0.0.110:80                Masq    1      0          0         
  -> 10.0.0.120:80                Masq    1      0          0         
  
  
#Client上curl Lvs的网关，根据权重显示RS-1和RS-2的内容
[root@ubuntu22 ~]$ curl 10.0.0.8
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.8
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 10.0.0.8
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.8
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 10.0.0.8
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.8
10.0.0.181 nginx rs-1
```



###### （2）DR模式单网段实现

![image-20240803212019703](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240803212019703.png)

网卡配置

- client eth0 网卡配置 192.168.10.100/24；   网关指向 192.168.10.200; 仅主机模式

- router eth0 网卡配置 192.168.10.200/24；  仅主机模式eth1 网卡配置 10.0.0.200/24；NAT模式

- LVS lo 网卡配置 VIP 10.0.0.100/32  

  eth0 网卡配置 DIP 10.0.0.8/24；网关指向 10.0.0.200；NAT 模式（VIP 收到192 网段 请求，此处网关用于逆向检查）

- RS1 lo 网卡配置 VIP 10.0.0.100/32        eth0 网卡配置 RIP 10.0.0.110/24；网关指向 10.0.0.200; NAT 模式

- RS2 lo 网卡配置 VIP 10.0.0.100/32        eth0 网卡配置 RIP 10.0.0.120/24；网关指向 10.0.0.200；NAT 模式

```bash
#client主机设置与nat相同（注意网关指向router节点）
#router主机设置，添加一块网卡，两块网卡一个设置为仅主机模式eth0，一个设置为NAT模式eth1
#eth0
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml
network:
    ethernets:
        eth0:
          addresses: [192.168.10.200/24]
          #dhcp4: true #是否自动分配ip
    version: 2
#eth1
root@ubuntu2 ~]$ cat /etc/netplan/eth1.yaml 
network:
  ethernets:
    eth1:
      addresses: [10.0.0.200/24]
  version: 2
#开启路由转发
[root@ubuntu2 ~]$ cat /etc/sysctl.conf
......
net.ipv4.ip_forward = 1
#查看
[root@ubuntu2 ~]$ sysctl -p
net.ipv4.ip_forward = 1

```

```bash
#lvs主机配置
#配置一块网卡，lo添加vip
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml
network:
    ethernets:
        eth0:
          addresses: [10.0.0.8/24]
          routes: [{to: default,via: 10.0.0.200}]  #网关指向Router
          # dhcp4: true
    version: 2
#lo 配置VIP
[root@ubuntu ~]# ifconfig lo:1 10.0.0.100 netmask 255.255.255.255
#查看
[root@ubuntu2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    ...
    inet 10.0.0.100/32 scope global lo:1
    ...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    ...
    inet 10.0.0.8/24 brd 10.0.0.255 scope global eth0
    ...
[root@ubuntu2 ~]$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.200      0.0.0.0         UG    0      0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

```bash
#RS-1、RS-2主机配置
#先下载net-tools插件 apt install net-tools
#网卡配置
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml
network:
  ethernets:
    eth0:
      addresses: [10.0.0.110/24] #RS-2 ip是10.0.0.120/24
      routes: [{to: default,via: 10.0.0.200}]  #网关指向Router
        # dhcp4: true
  version: 2
#lo文件配置（RS-1、RS-2、Lvs的 VIP相同）
[root@ubuntu ~]# echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
[root@ubuntu ~]# echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce

[root@ubuntu ~]# echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore 
[root@ubuntu ~]# echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce

[root@ubuntu ~]# ifconfig lo:1 10.0.0.100 netmask 255.255.255.255
#查看
[root@ubuntu2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
   ......
    inet 10.0.0.100/32 scope global lo:1
   ......
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    ......
    inet 10.0.0.110/24 brd 10.0.0.255 scope global eth0
    ......
[root@ubuntu2 ~]$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.200      0.0.0.0         UG    0      0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

```bash
#LVS主机配置ipvsadm
[root@ubuntu2 ~]$ ipvsadm -A -t 10.0.0.100:80 -s wrr        #wrr 设置加权轮询算法
[root@ubuntu2 ~]$ ipvsadm -a -t 10.0.0.100:80 -r 10.0.0.120 -g -w 2  #权重是2  -g DR模式
[root@ubuntu2 ~]$ ipvsadm -a -t 10.0.0.100:80 -r 10.0.0.110 -g -w 1  #权重是1
#查看
[root@ubuntu2 ~]$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.100:80 wrr
  -> 10.0.0.110:80                Route   1      0          0         
  -> 10.0.0.120:80                Route   2      0          0    
```

```bash
#client主机测试
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
#RS-2 120权重是2，RS-2出现2次 RS-1出现1次
```



###### （3）DR模式多网段实现

DR单网段容易暴露后端RS服务器地址多网段不会·

![image-20240804144225188](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240804144225188.png)

**配置说明**

- client配置与单网段相同

- router  eth0 网卡配置 192.168.10.200/24；仅主机模式

   			 eth1 网卡配置 10.0.0.200/24；NAT模式

   			 eth1:1 网卡配置 172.16.0.200/24；NAT模式

- RS1   lo 网卡配置 VIP 172.16.0.100/32    eth0 网卡配置 RIP 10.0.0.110/24； 网关指向 10.0.0.200; NAT 模式

  RS2   lo 网卡配置 VIP 172.16.0.100/32    eth0 网卡配置 RIP 10.0.0.120/24； 网关指向 10.0.0.200；NAT 模式

- LVS lo 网卡配置 VIP 172.16.0.100/32 

   eth0 网卡配置 DIP 10.0.0.8/24；网关指向 10.0.0.200；NAT 模式（此处网关不能少）

```bash
#client与单网段client主机配置相同
#route eth0与单网段配置相同 （仅主机）
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml 
network:
    ethernets:
        eth0:
          addresses: [192.168.10.200/24]
            #dhcp4: true
    version: 2
#eth1
#增加172.16.0.200/24的VIP（vip由单网段的10.0.0.200改为172.16.0.20）
[root@ubuntu2 ~]$ cat /etc/netplan/eth1.yaml 
network:
  ethernets:
    eth1:
      addresses: 
       - 10.0.0.200/24 #删除也可
       - 172.16.0.200/24: 
          label: "eth1:1"
  version: 2
[root@ubuntu2 ~]$ netplan apply 
#查看
[root@ubuntu2 ~]$ ip a
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    ...
    inet 192.168.10.200/24 brd 192.168.10.255 scope global eth0
    ...
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    ...
    inet 10.0.0.200/24 brd 10.0.0.255 scope global eth1
    ...
    inet 172.16.0.200/24 brd 172.16.0.255 scope global eth1:1
    ...
#开启路由转发
[root@ubuntu ~]# sysctl -p
net.ipv4.ip_forward = 1
#如果等于0   echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf
```

```bash
#Lvs配置
[root@ubuntu2 ~]$ cat /etc/netplan/50-cloud-init.yaml 
network:
    ethernets:
        eth0:
          addresses: [10.0.0.8/24]
          routes: [{to: default,via: 10.0.0.200}]
          # dhcp4: true
    version: 2
 [root@ubuntu2 ~]$ netplan apply
 #查看
 [root@ubuntu2 ~]$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.200      0.0.0.0         UG    0      0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.10.0    0.0.0.0         255.255.255.0   U     0      0        0 eth1
#Lvs、Rs-1、Rs-2的 eth0配置均与单网段配置相同，VIP由10.0.0.200/24改为172.16.0.200/24
```

```bash
#客户端主机测试
#此时vip未更改，client主机可curl通Rs-1、Rs-2
#更改RS-1、Rs-2的lo（更改vip）
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
[root@ubuntu2 ~]$ echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore 
[root@ubuntu2 ~]$ echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
#先删除单网段配置的VIP lo:1
[root@ubuntu2 ~]$ ifconfig lo:1 down; ifconfig lo:1 172.16.0.100 netmask 255.255.255.255

#Lvs主机配置VIP
[root@ubuntu2 ~]$ ifconfig lo:1 172.16.0.100 netmask 255.255.255.255
#Lvs上配置转发规则
[root@ubuntu2 ~]$ ipvsadm -A -t 172.16.0.100:80 -s rr #rr是轮询算法
[root@ubuntu2 ~]$ ipvsadm -a -t 172.16.0.100:80 -r 10.0.0.110 -g -w 1 
[root@ubuntu2 ~]$ ipvsadm -a -t 172.16.0.100:80 -r 10.0.0.120 -g -w 2 #权重为2
#查看
[root@ubuntu2 ~]$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.16.0.100:80 rr
  -> 10.0.0.110:80                Route   1      0          0         
  -> 10.0.0.120:80                Route   2      0          0  
  
  
#client主机测试（rr是轮询算法，与权重无关）
[root@ubuntu22 ~]$ curl 172.16.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 172.16.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 172.16.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 172.16.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 172.16.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 172.16.0.100
10.0.0.120 nginx rs-2

```

###### （4）TUN模式实现

![image-20240804154152805](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240804154152805.png)

- client      eth0 网卡配置 192.168.10.100/24；网关指向 192.168.10.200; 仅主机模式

- router     eth0 网卡配置 192.168.10.200/24；仅主机模式

  ​                 eth1 网卡配置 10.0.0.200/24；NAT模式

- LVS          tunl0 网卡配置 VIP 10.0.0.100/32  

  ​				 eth0 网卡配置 DIP 10.0.0.8/24；网关指向 10.0.0.200；NAT 模式（此处网关不能少）

- RS1          tunl0 网卡配置 VIP 10.0.0.100/32  

  ​				 eth0 网卡配置 RIP 10.0.0.110/24；网关指向 10.0.0.200; NAT 模式

- RS2          tunl0 网卡配置 VIP 10.0.0.100/32  

  ​				 eth0 网卡配置 RIP 10.0.0.120/24；网关指向 10.0.0.200；NAT 模式

```bash
#client、Router、LVS Rs-1、2  主机配置与DR单网段相同

#Rs-1/Rs-2 主机配置VIP
#删除以前配置的VIP
[root@ubuntu2 ~]$ ifconfig lo:1 down
#设置新的VIP tun10
[root@ubuntu2 ~]$ ifconfig tuifconfig tunl0 10.0.0.100 netmask 2

#修改内核参数，禁用VIP 的 ARP广播和应答
#tun模式中，如果RS与LVS不在同一物理网络可以不禁止
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/al
[root@ubuntu2 ~]$ echo 2 > /proc/sys/net/ipv4/conf/a

[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/t
[root@ubuntu2 ~]$ echo 2 > /proc/sys/net/ipv4/conf/t

#禁止反向检测报文是否能回去
[root@ubuntu2 ~]$ echo 0 > /proc/sys/net/ipv4/conf/al
[root@ubuntu2 ~]$ echo 0 > /proc/sys/net/ipv4/conf/tu

#查看
[root@ubuntu2 ~]$ ip a s tunl0
3: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state 
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 10.0.0.100/32 scope global tunl0
       valid_lft forever preferred_lft forever

#LVS设置VIP （关闭以前的vip）
[root@ubuntu2 ~]$ ifconfig lo:1 down
[root@ubuntu2 ~]$ ifconfig tunl0 10.0.0.100 netmask 255.255.255.255

#启用 tunl0 网卡会自动加载IPIP模块
[root@ubuntu2 ~]$ lsmod | grep ipip
ipip                   20480  0
tunnel4                12288  1 ipip
ip_tunnel              32768  1 ipip

#Lvs配置转发规则
[root@ubuntu2 ~]$ ipvsadm -A -t 10.0.0.100:80 -s rr
[root@ubuntu2 ~]$ ipvsadm -a -t 10.0.0.100:80 -r 10.0.0.110:80 -i  # -i tun模式
[root@ubuntu2 ~]$ ipvsadm -a -t 10.0.0.100:80 -r 10.0.0.120:80 -i
#查看
[root@ubuntu2 ~]$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn      
TCP  10.0.0.100:80 rr
  -> 10.0.0.110:80                Tunnel  1      0          0         
  -> 10.0.0.120:80                Tunnel  1      0          0  
  
#client curl测试
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.181 nginx rs-1

```

##### 6、防火墙标签实现集群组

```bash
#基于TUN模式环境
#client，Router，RS节点配置不变（RS必须有nginx，apache端口分别为80、88）

##使用 iptables 打标签，目标为 10.0.0.100，协议为 tcp，目标端口为 80 或 88 的报文，打个标签，值为 10
[root@ubuntu2 ~]$ iptables -t mangle -A PREROUTING -d 10.0.0.100 -p tcp -m multiport --dports 80,88 -j MARK --set-mark 10
#查看防火墙
[root@ubuntu2 ~]$ iptables -t mangle -vnL PREROUTING
Chain PREROUTING (policy ACCEPT 20 packets, 1416 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MARK       6    --  *      *       0.0.0.0/0            10.0.0.100           multiport dport
                       #port 6 是tcp协议
#清空原有集群
[root@ubuntu2 ~]$ ipvsadm -C
#使用防火墙标签进行调度
[root@ubuntu2 ~]$ ipvsadm -A -f 10 -s rr
[root@ubuntu2 ~]$ ipvsadm -a -f 10 -r 10.0.0.110 -i
[root@ubuntu2 ~]$ ipvsadm -a -f 10 -r 10.0.0.120 -i
#查看
[root@ubuntu2 ~]$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
FWM  10 rr
  -> 10.0.0.110:0                 Tunnel  1      0          0         
  -> 10.0.0.120:0                 Tunnel  1      0          0  
  
#client进行curl测试
#同时调度时把80，88 当作一个集群调度
#非防火墙标签模式把80 88分开调度，分别做RR算法均衡
[root@ubuntu22 ~]$ curl 10.0.0.100; curl 10.0.0.100:88
10.0.0.181 nginx rs-1   
10.0.0.120 apache Rs-2  
[root@ubuntu22 ~]$ curl 10.0.0.100; curl 10.0.0.100:88
10.0.0.181 nginx rs-1
10.0.0.120 apache Rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.181 nginx rs-1
[root@ubuntu22 ~]$ curl 10.0.0.100
10.0.0.120 nginx Rs-2
[root@ubuntu22 ~]$ curl 10.0.0.100:88
10.0.0.181 apache rs-1
[root@ubuntu22 ~]$ curl 10.0.0.100:88
10.0.0.120 apache Rs-2

```

##### 7、LVS 实现持久连接

LVS 持久连接，在此配置项生效时间内，将来自于同一个客户端的连接（同一个CIP）转发到一台相同的 后端 RS 服务器上，单位为秒，默认值是 360S

```bash
#查看当前超时时长值，单位秒
[root@ubuntu ~]# ipvsadm -L --timeout
Timeout (tcp tcpfin udp): 900 120 300

#修改默认超时时长，便于测试
[root@ubuntu ~]# ipvsadm --set 20 20 20
[root@ubuntu ~]# ipvsadm -L --timeout
Timeout (tcp tcpfin udp): 20 20 20

#设置集群超时时长
[root@ubuntu ~]# ipvsadm -E -t 10.0.0.100:80 -s rr -p 10
[root@ubuntu ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.100:80 rr persistent 10
  -> 10.0.0.110:80               Route   1      0          0         
  -> 10.0.0.120:80               Route   1      0          0  
#客户端测试

[root@ubuntu ~]# curl 10.0.0.100;
this page from rs-2 nginx
#LVS上查看，在持久连接有效期内，同一个客户端请求会被调度到同一台后端 RS 服务器上
#如果 -p 设置的时长小于连接超时时长，则 ASSURED 记录超时后，如果上面非活动连接状态还在，则会自动续命60S


#修改持久连接时长，大于请求超时时长
[root@ubuntu ~]# ipvsadm -E -t 10.0.0.100:80 -s rr -p 30
#非活动连接消失后持久连接也消失

```

