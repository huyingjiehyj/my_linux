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



##### 2、安装

```bash
#lsb_release -a  查看ubuntu版本号
#进入  https://www.zabbix.com/cn/download 根据要求选择zabbix版本，等
#根据提示进行下载

[root@ubuntu2 ~]$ wget https:wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-6+ubuntu24.04_all.deb
[root@ubuntu2 ~]$ dpkg -i zabdpkg -i zabbix-release_6.0-6+ubuntu24.04_all.deb

#会生成一个源的配置文件
[root@ubuntu ~]# cat /etc/apt/sources.list.d/zabbix.list 
# Zabbix main repository
deb https://repo.zabbix.com/zabbix/6.0/ubuntu noble main
deb-src https://repo.zabbix.com/zabbix/6.0/ubuntu noble main

[root@ubuntu2 ~]$ apt update
```

