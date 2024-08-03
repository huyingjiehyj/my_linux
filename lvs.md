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

