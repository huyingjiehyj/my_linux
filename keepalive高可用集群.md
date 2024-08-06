#### keepalive高可用集群

##### 1、VRRP 协议

- VRRP是一种网络协议：允许多个路由器作为一个虚拟路由器组 （VRID）工作，实现冗余和故障转移，提高网络容错性

- 特性·：

  - 冗余路由器组，允许多个路由器配置为一个虚拟路由组，一主（master）多备（backup）

  - 虚拟路由器标识（VRID）每个 VRRP 路由器组都有一个唯一的 VRID，用于区分不同的组

  - 主备切换:master失效，backup会接管工作，切换是透明的，终端设备无感知

  - 优先级： 每个 VRRP 路由器都有一个预先配置的优先级。优先级最高的路由器将成为主路由器，master失效，backup优先级高的顶替

  - 健康检查：VRRP 路由器会定期发送通告消息来确认它们的健康状态。如果其他路由器在一定时间内没有收到主路由器通告的消息，则假定主路由器失效

    

##### 2、keepalive安装

```bash
#rocky安装
[root@rocky ~]# yum install keepalived

[root@rocky ~]# which keepalived 
/usr/sbin/keepalived

#Ubuntu安装2404
[root@ubuntu ~]# apt update;apt install keepalived -y
#会自动安装 ipvsadm
[root@ubuntu ~]# which ipvsadm
/usr/sbin/ipvsadm
#查看版本号
[root@ubuntu ~]# keepalived -v

#默认没有启动，缺配置文件（将keepalived.conf.sampl 改为 keepalived.conf）
[root@ubuntu ~]# cp /etc/keepalived/keepalived.conf.sampl /etc/keepalived/keepalived.conf

#将网卡名称改为eth0 或 把配置文件中eth0改为ens33 才能启动成功

#Keepalive内嵌防火墙策略，无法ping通keepalive配置文件添加的ip
#无法用iptables查看但可以用 nft 查看

[root@ubuntu ~]# nft list ruleset

table ip keepalived {
 set vips {
 type ipv4_addr
 elements = { 192.168.200.16, 192.168.200.17,
     192.168.200.18 }
 }
 ...
#nft 清空
[root@ubuntu ~]# nft flush ruleset
[root@ubuntu ~]# nft list ruleset

```

##### 3、keepalive高可用



前置条件

- 各节点操作系统保持一致 

- 各节点时区，时间统一 

- 各节点网卡修改成传统设备命名形式，如 eth0，eth1，等 

- 各节点关闭防火墙

  ```bash
  #修改网卡名
  [root@ubuntu22 ~]$ cat /etc/default/grub
  GRUB_CMDLINE_LINUX=" net.ifnames=0"
  
  [root@ubuntu22 ~]$ vim /etc/netplan/50-cloud-init.yaml
  ...
          eth0:
  ...
  
  [root@ubuntu22 ~]$ grub-mkconfig -o /boot/grub/grub.cfg; reboot
  ```

  

![image-20240805194112023](C:/Users/26914/AppData/Roaming/Typora/typora-user-images/image-20240805194112023.png)

Keepalived - 1 (master)     NAT：10.0.0.216/24，10.0.0.123/24（VIP） 

​                                              仅主机：192.168.10.8/24   用作心跳检测

Keepalived - 2 (backup)     NAT：10.0.0.217/24 ubuntu2404 keepalived，

l                                              仅主机：192.168.10.18/24   用作心跳检测

web server - 1         NAT：10.0.0.218/24 

​                                   lo：10.0.0.123/32（VIP）   

web - server -2        NAT：10.0.0.219/24 

​                                   lo：10.0.0.123/32（VIP）   

```bash
#k1配置
#eth0
[root@ubuntu22 ~]$ vim /etc/netplan/50-cloud-init.yaml
network:
    ethernets:
        eth0:
          #dhcp4: true
          addresses: [10.0.0.216/24]
          routes: [{to: default,via: 10.0.0.2}]
          nameservers:
            addresses: [10.0.0.2,223.5.5.5]
    version: 2
 #eth1（仅主机）
[root@ubuntu22 ~]$ vim /etc/netplan/eth1.yaml 
network:
 ethernets:
   eth1:
     addresses: [192.168.10.8/24]
 version: 2
 
[root@ubuntu22 ~]$ netplan apply

#确认网卡（ip a）
#安装keepalive
#配置master节点
[root@ubuntu22 ~]$ vim /etc/keepalived/keepalived.conf

global_defs {
  router_id k216                  #当前节点ID k216
  vrrp_mcast_group4 226.6.6.6     #广播地址 236.6.6.6

}

vrrp_instance VI_1{               #VRRP
  state MASTER                    #master节点
  interface eth1                  #eth1用于心跳检测
  virtual_router_id 66            #集群id是 66
  priority 100                    #当前节点优先级 100
  advert_int 3                    #3s 发送一个心跳包
  authentication{                 #同一集群认证凭证
      auth_type PASS
      auth_pass 123456

  }
  virtual_ipaddress {

      10.0.0.123/24 dev eth0      #vip是10.0.0.123 在eth0 上
  }

}
#重启服务
[root@ubuntu22 ~]$ systemctl restart keepalived.service
#VIP 己加到 eth0上（如果没有试一下iptabes -F）
#在 windows 中测试   ping 10.0.0.123 能ping通
#抓 VVRP 包，当前主机己经在向广播IP发送数据了

[root@ubuntu ~]# tcpdump vrrp -nn -i eth1
...
08:04:36.220343 IP 192.168.10.8 > 226.6.6.6: VRRPv2, Advertisement, vrid 66, 
prio 100, authtype simple, intvl 3s, length 20

```

```bash
#k2配置
#eth0
[root@ubuntu2 ~]$ vim /etc/netplan/50-cloud-init.yaml 
network:
    ethernets:
        eth0:
          #dhcp4: true
          addresses: [10.0.0.217/24]
          routes: [{to: default,via: 10.0.0.2}]
          nameservers:
            addresses: [10.0.0.2,233.5.5.5]
    version: 2
#eth1
[root@ubuntu2 ~]$ vim /etc/netplan/eth1.yaml 
network:
  ethernets:
    eth1:
      addresses: [192.168.10.18/24]
  version: 2
[root@ubuntu2 ~]$ netplan apply
#确认网卡
#安装keepalive
#配置slave节点
[root@ubuntu2 ~]$ vim /etc/keepalived/keepalived.conf
global_defs{
  router_id k217                  #当前节点ID k216
  vrrp_mcast_group4 226.6.6.6
}
vrrp_instance VI_1{
  state BACKUP
  interface eth1                  #eth1 用于心跳检测
  virtual_router_id 66            #集群id 66
  priority 80                     #优先级80
  advert_int 3                    #3s 发送一个心跳包
  authentication {
      auth_type PASS
      auth_pass 123456
  }
  virtual_ipaddress {
      10.0.0.123/24 dev eth0    #vip
  }
}
#重启
[root@ubuntu ~]# systemctl restart keepalived.service 
#ip a当前主机没有VIP（master vip在eth0上）

#只有Master 节点在发送数据报文
[root@ubuntu ~]# tcpdump vrrp -nn -i eth1
...
11:31:43.373207 IP 192.168.10.8 > 226.6.6.6: VRRPv2, Advertisement, vrid 66, 
prio 100, authtype simple, intvl 3s, length 20
```

**测试VIP漂移（master停止服务，slave变成master）**

```bash
#Master 节点
[root@ubuntu ~]# hostname -I
10.0.0.216 10.0.0.123 192.168.10.8
#Slave 节点
[root@ubuntu ~]# hostname -I
10.0.0.217 192.168.10.18
#第三台主机长时间ping10.0.0.123（VIP）
#关闭master节点的keepalive
[root@ubuntu ~]# systemctl stop keepalived.service
#两个节点分别hostname -I 发现VIP到slave节点了

#systemctl status keepalive.service 发现slave节点变成了master节点
#ping未中断
#抓包，slave节点发送广播
[root@ubuntu ~]# tcpdump -n -i eth1
...
11:48:03.827202 IP 192.168.10.18 > 226.6.6.6: VRRPv2, Advertisement, vrid 66, 
prio 80, authtype simple, intvl 3s, length 20
...
#k1 服务起来后，VIP还会漂回来，k1从slave变为master，k2变为slave

```

**为不同项目设置独立配置文件**

```bash
#创建目录
[root@ubuntu2 ~]$ mkdir /etc/keepalived/conf.d

[root@ubuntu22 ~]$ cat /etc/keepalived/keepalived.conf
global_defs {
  router_id k216
  vrrp_mcast_group4 226.6.6.6

}

include /etc/keepalived/conf.d/*.conf #引用conf.d所有的conf文件
#每个项目一个配置文件
[root@ubuntu22 ~]$ cat /etc/kecat /etc/keepalived/conf.d/www.m99-magedu.com.conf 
vrrp_instance VI_1{
  state MASTER
  interface eth1
  virtual_router_id 66
  priority 100
  advert_int 3
  authentication{
      auth_type PASS
      auth_pass 123456
  
  }
  virtual_ipaddress {

      10.0.0.123/24 dev eth0
  }
  
}
#测试
[root@ubuntu22 ~]$ hostname -I
10.0.0.216 10.0.0.123 192.168.10.8

[root@ubuntu ~]# tree /etc/keepalived/
/etc/keepalived/
├── conf.d
│   └── www.m99-magedu.com.conf
├── keepalived.conf

```

**集群脑裂现象分析**

脑裂：在 keepalived 集群中，同一时间，VIP 存在于一个以上的节点上，就是脑裂

```bash
#判断当前有几个VIP

[root@rocky ~]# arping 10.0.0.123
ARPING 10.0.0.123 from 10.0.0.213 ens160
Unicast reply from 10.0.0.123 [00:0C:29:6B:F1:9B]  2.658ms
Unicast reply from 10.0.0.123 [00:0C:29:6B:F1:9B]  5.449ms

#在backup节点添加一个vip 10.0.0.123   #出现脑裂
[root@ubuntu ~]# ip a a 10.0.0.123/24 dev eth0
#第三台主机上测试当前环境中有两个VIP
[root@rocky ~]# arping 10.0.0.123 -c1
ARPING 10.0.0.123 from 10.0.0.213 ens160
Unicast reply from 10.0.0.123 [00:0C:29:6B:F1:9B]  1.261ms
Unicast reply from 10.0.0.123 [00:0C:29:C1:F9:8B]  1.498ms

#在BACKUP 节点删除VIP
[root@ubuntu ~]# ip a d 10.0.0.123/24 dev eth0

#第三台主机上测试再次测试，只有一个VIP
[root@rocky ~]# arping 10.0.0.123 -c1
ARPING 10.0.0.123 from 10.0.0.213 ens160
Unicast reply from 10.0.0.123 [00:0C:29:6B:F1:9B]  1.356ms
Sent 1 probes (1 broadcast(s))
Received 1 response(s)




#防火墙配置导致脑裂
##在BACKUP 节点上设置防火墙，使该节点上的广播地址无法收到数据包 
[root@ubuntu ~]# iptables -A INPUT -d 226.6.6.6 -j DROP
#该节点也获得了VIP
[root@ubuntu ~]# hostname -I
10.0.0.217 10.0.0.123 192.168.10.18 


#网线断开导致脑裂
#keepalive配置出错导致脑裂 #route id 与 MASTER 节点不一致、网络设置配错、#密码不一致

```

**建立独立日志**

```bash
#当前日志写在系统日志中
[root@ubuntu ~]# tail /var/log/syslog 
#keepalived 中和 log 相关的选项
[root@ubuntu ~]# keepalived -h 
#添加选项，日志设置为 local6
[root@ubuntu ~]# cat /etc/default/keepalived
local6.* /var/log/keepalived.log #添加此行
#重启服务

[root@ubuntu ~]# systemctl restart rsyslog.service; systemctl restart keepalived.service;
#产生了日志
[root@ubuntu ~]# ls /var/log/keepalived.log
/var/log/keepalived.log


```

**抢占模式和非抢占模式**

- 抢占模式：Master 节点不可用之后，VIP 会漂移到 Backup 节点，当Master 节点再次上线后，VIP 

还是会再次回到 Master 节点，

- 非抢占式：两个节点都设置backup，作为master节点的优先级更高，

```bash
[root@ubuntu ~]# cat /etc/keepalived/conf.d/www.m99-magedu.com.conf 
vrrp_instance www.m99-magedu.com {
   state BACKUP                 #非抢占式都是BACKUP
   interface eth1
   virtual_router_id 66
   priority 100                             # 但MASTER节点优先级要更高
   advert_int 3
   nopreempt                                    #显式指定非抢占式
   authentication {
       auth_type PASS
       auth_pass 123456
   }
   virtual_ipaddress {
        10.0.0.123/24 dev eth0
   }
}

#SLAVE 节点配置
[root@ubuntu ~]# cat /etc/keepalived/conf.d/www.m99-magedu.com.conf 
vrrp_instance www.m99-magedu.com {
   state BACKUP
   interface eth1
   virtual_router_id 66
   priority 80
   advert_int 3
   authentication {
       auth_type PASS
       auth_pass 123456
   }
   virtual_ipaddress {
        10.0.0.123/24 dev eth0
   }
}
#停止master节点keepalive服务，vip到slave节点，但是恢复后，vip仍在slave节点，slave变master
```

抢占延时模式

抢占延时模式（preempt_delay）：优先级高的主机恢复后，不会立即抢回VIP，而是延时一段时间（默 认300S）再抢回 VIP，但如果优先级低的主机不可用后，会立即获得 VIP，而不会延时，此模式中不支 持 vrrp_strict

```bash
#MASTER 节点配置

[root@ubuntu ~]# cat /etc/keepalived/conf.d/www.m99-magedu.com.conf

vrrp_instance www.m99-magedu.com {
   state BACKUP             #BACKUP
   interface eth1
   virtual_router_id 66
   priority 100             #优先级更高
   advert_int 3
   preempt_delay 15          #延时15S，默认300S
   authentication {
       auth_type PASS
       auth_pass 123456
   }
   virtual_ipaddress {
        10.0.0.123/24 dev eth0
   }
}

#SLAVE 节点配置

[root@ubuntu ~]# cat /etc/keepalived/conf.d/www.m99-magedu.com.conf

vrrp_instance www.m99-magedu.com {
   state BACKUP #BACKUP
   interface eth1
   virtual_router_id 66
   priority 80 #优先级低
   advert_int 3
   authentication {
       auth_type PASS
       auth_pass 123456
   }
   virtual_ipaddress {
        10.0.0.123/24 dev eth0
   }
}
```

###### keepalived实现lvs高可用

```bash
#后台RS-1/RS-2配置
#nginx环境
[root@ubuntu2 ~]$ vim /var/www/html/index.html 
RS-1 nginx #RS-2 nginx
[root@ubuntu2 ~]$ vim /var/www/html/ping.html 
pong
#重启nginx环境
[root@ubuntu2 ~]$ systemctl restart nginx.service 
#配置VIP地址 并禁止VIP的ARP请求和响应
[root@ubuntu2 ~]$ ip a a 10.0.0.123/32 dev lo
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/all/arp_announce
[root@ubuntu2 ~]$ echo 1 > /proc/sys/net/ipv4/conf/lo/arp_announce
#查看
[root@ubuntu2 ~]$  ip a 

```

```bash
#10.0.0.216/10.0.0.217  keeplive节点设置
[root@ubuntu22 ~]$ cat /etc/keepalived/keepalived.conf
global_defs {
  router_id k216
  vrrp_mcast_group4 226.6.6.6
}
include /etc/keepalived/conf.d/*.conf #引用路径下conf.d中的配置文件


#写配置文件和ipvsadm规则
[root@ubuntu22 ~]$ cat /etc/keepalived/conf.d/www.m99-magedu.com.conf 
vrrp_instance www.m99-magedu.com{
  state MASTER                         #10.0.0.217 是BACKUP
  interface eth1
  virtual_router_id 66
  priority 100                         #优先级10.0.0.217是80
  advert_int 1
  unicast_src_ip 192.168.10.8
  unicast_peer{
    192.168.10.18
  }
  authentication{
      auth_type PASS
      auth_pass 123456
  }
  virtual_ipaddress {
      10.0.0.123/24 dev eth0
  }
}

virtual_server 10.0.0.123 80{
  delay_loop 3 #3s检查一次后端服务器
  lb_algo rr    #轮询算法
  lb_kind DR    #DR模型不支持端口映射
  protocol TCP   #TCP协议
  
  real_server 10.0.0.218 80 {  #RS-1
     weight 1   #权重
     HTTP_GET { #后端主机检测
       url {
             path /ping.html
             status_code 200
       }
       connect_timeout 1  #超时时长
       retry 3            #重试次数
       delay_before_retry 1  #重试前等待时长
     }

  }
  real_server 10.0.0.219 80 { #RS-2
       weight 1
       HTTP_GET {
          url {
             path /ping.html
             status_code 200
          }
           connect_timeout 1
           retry 3
           delay_before_retry 1
       }
  }

}
#重启服务
[root@ubuntu22 ~]$ systemctl restart keepalived.service 
#查看ipvsadm
root@ubuntu22 ~]$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.123:80 rr
  -> 10.0.0.162:80                Route   1      0          0         
  -> 10.0.0.181:80                Route   1      0          0 
  
```

```bash
#RS主机上查看健康检查请求
[root@ubuntu2 ~]$ tail /var/log/nginx/access.log 
10.0.0.216 - - [06/Aug/2024:12:10:49 +0000] "GET /ping.html HTTP/1.0" 200 5 "-" "KeepAliveClient"
10.0.0.217 - - [06/Aug/2024:12:10:50 +0000] "GET /ping.html HTTP/1.0" 200 5 "-" "KeepAliveClient"
10.0.0.216 - - [06/Aug/2024:12:10:52 +0000] "GET /ping.html HTTP/1.0" 200 5 "-" "KeepAliveClient"
10.0.0.217 - - [06/Aug/2024:12:10:53 +0000] "GET /ping.html HTTP/1.0" 200 5 "-" "KeepAliveClient"
10.0.0.216 - - [06/Aug/2024:12:10:55 +0000] "GET /ping.html HTTP/1.0" 200 5 "-" "KeepAliveClient"
10.0.0.217 - - [06/Aug/2024:12:10:56 +0000] "GET /ping.html HTTP/1.0" 200 5 "-" "KeepAliveClient"



#客户端测试
[root@10 ~]$ cat /etc/hosts
10.0.0.123 www.m99-magedu.com
#curl www.m99-magedu.com
```

###### 双主模型实现

```bash
#RS-1/RS-2 节点不变，只需要将 master改为BACKUP，主节点比备用节点优先级高
[root@ubuntu22 ~]$ cat /etc/keepalived/conf.d/www.m99-magedu.com.conf 
vrrp_instance www.m99-magedu.com{
  state BACKUP                       
  interface eth1
  virtual_router_id 66
  priority 100                         #优先级10.0.0.217是80
  advert_int 1
  unicast_src_ip 192.168.10.8
  unicast_peer{
    192.168.10.18
  }
```

**实现HTTPS调度**（nginx证书）
