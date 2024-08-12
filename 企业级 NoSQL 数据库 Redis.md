#### 企业级 NoSQL 数据库 Redis

数据库分为关系型数据库和nosql数据库

- 关系型数据库：建立在关系模型上的数据库，借助集合代数等数学概念的方法处理数据库中的数据。
- 非关系型数据库： 适用关系型数据库时就用关系型数据库，不适用时就考虑更合适的方式存储。

##### 1、关系型数据库RDBMS和nosql比较



|              | RDBMS                                                        | NOSQL                                                        |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 数据类型     | 用表格表示数据，具有严格模式                                 | 有多种数据模型（文档，键值对，列存储，图）                   |
|              | 表与表之间有外键建立关系                                     | 没有固定模式，数据模型灵活                                   |
|              | 适合结构化数据，适用于事务处理和关系密集型的应用             | 适合半结构化和非结构化数据，适用数据结构频繁变化的应用       |
| 查询语言     | 结构化语言（SQL）                                            | 各种NOSQl有自己的结构化语言和api                             |
|              | SQ标准化程度高，易于学习                                     | 查询功能因数据库类型不同而异，灵活性高                       |
| 可扩展性     | 通常是垂直扩展（堆积器增加性能）                             | 水平扩展（通过增加服务器处理更多的服务请求）                 |
|              | 水平扩展困难                                                 | 适合大规模分布式系统                                         |
| 一致性和事务 | 遵循ACID原则（原子性，一致性，隔离性，持久性）原则，确保数据一致性和可靠性 | 强调CAP定理（一致性，可用性，分区容忍性）中的可用性和分区容忍性 |
|              | 适合需要强一致性和复杂事务的应用                             | 支持最终一致性，也有一些NOSQL（MongoDB）支持事务             |
| 性能         | 处理复杂查询和事务时性能较好                                 | 在读写操作频繁，高吞吐量，低延迟表现优越                     |
|              | 适合读写操作少查询复杂度高的应用                             | 适合快速读写，还来数据处理应用                               |
| 数据完整性   | 通过外键，唯一性约束等机制确保数据完整性                     | 数据完整性主要由应用程序层来管理                             |
|              | 更适合需要严格数据完整性和约束的应用                         | 更灵活，数据完整性一致性需小心处理                           |

总结：

RDBMS：优点：数据一致性强，事务支持，标准化查询语言，数据完整性好

​                 缺点; 扩展有限，灵活性差，在大规模数据处理中的性能瓶颈

NOSQL： 优点：高扩展性，灵活的数据模型，高性能读写，适用大规模数据处理

​                  缺点： 一致性和事务支持相对较弱，查询语言不统一，数据完整性依赖应用程序



**CAP定理：**

- C:   一致性： 所有节点在同一时间具有相同的数据视图（强一致性）
- A：可用性： 所有节点保持高可用性（及时响应，不能阻塞）
- P：分区容忍性：系统能在网络分区情况下继续运行。分区指节点被分为多个孤立子系统，无法相互通信（脑裂）

**一个数据分布式系统不可能同时满足C和A和P这3个条件**。常实现AP都会保证最终一致性



**BASE理论**

- 基本可用：系统在故障时，保证系统依然可用

- 软状态：允许系统 在不同节点的数据副本上存在数据延时

- 最终一致性：不保证用户 什么时候能看到更新完成后的数据，但是终究会看到的

  

**NOSQL数据分类：**

列存储、文档存储、key-value存储（redis）、图存储、对象存储、xml数据库





##### 2、Redis

Redis 基于ANSI C 语言语言编写的 key-value 数据库



**Redis 中的单线程**

redis是单进程多线程的模型，只有一个线程处理客户端发送来的命令，每个命令都要排队执行，而不 是说 redis 只有一个线程

redies使用单线程处理所有客户端请求，所有命令按顺序执行，不并行处理（不是说Redis只有一个线程）

优点：简单，无需处理多线程同步和锁问题

​            高效：避免上下文切换和锁争用

​           低延迟

缺点： cpu瓶颈：单线程在cpu密集型任务下可能成为瓶颈，无法发货服务器多核心的作用

​            慢查询：长时间查询会阻塞请求



**缓存穿透，缓存击穿，缓存雪崩**

**缓存穿透**：查询一个不存在的数据，缓存中没有这个数据的缓存，也不会缓存这个数据的请求，导致每次请求都会直接穿透缓存，查询数据库。



解决方法：布置过滤器，数据不存在直接返回、

​                  缓存空值：对于不存在的数据把空结果缓存设置较短的过期时间



**缓存击穿**：某个热点数据失效后，大量请求同时到达缓存，导致请求大量打到数据库上，造成数据库压力过大



解决方法：互斥锁：缓存失效时，通过加锁方式，只有一个线程去加载数据，其他线程等待

​                   提前更新：缓存即将失效时，提前异步更新缓存

​                  设置热点事件永远不过期



**缓存雪崩**：服务器大量缓存在同一时间失效或缓存服务器宕机，大量请求打到数据库上



解决方法： 缓存失效时间设置随机化

​                    多级缓存;通过本地缓存和远程缓存多级缓存策略，降低缓存失效对数据库的影响

​                    限流和降级：发现数据库负载过高时，通过限流和服务器降级措施，保护数据库不被过载







##### 3、Redis安装配置

```bash
#ubuntu
[root@ubuntu2 ~]$ apt update;apt install redis
#下载了三个包
[root@ubuntu2 ~]$ dpkg -l redis*
...
ii  redis          5:7.0.15-1build2 all          Persistent key-value database with network interface (metapackage)
ii  redis-server   5:7.0.15-1build2 amd64        Persistent key-value database with network interface
ii  redis-tools    5:7.0.15-1build2 amd64        Persistent key-value database with network interface (client)
#查看状态
[root@ubuntu2 ~]$ systemctl status redis-server.service 
#监听本地6379端口
[root@ubuntu2 ~]$ ss -tnlp | grep redis
LISTEN 0      511        127.0.0.1:6379      0.0.0.0:*    users:(("redis-server",pid=2269,fd=6))             
LISTEN 0      511            [::1]:6379         [::]:*    users:(("redis-server",pid=2269,fd=7))
#client连接服务器
[root@ubuntu2 ~]$ redis-cli
127.0.0.1:6379>

#查看当前生效配置
[root@ubuntu2 ~]$ cat /etc/redis/redis.conf | grep -Ev "^#|^$

```

```bash
#修改配置让远程主机连接
[root@ubuntu2 ~]$ vim /etc/redis/redis.conf
#bind 127.0.0.1 -::1 默认只监听本机ip
bind 0.0.0.0         #修改为监听所有本机ipv4地址

#protected-mode yes   #保护模式 远程可用连接无法获取数据
protected-mode no     #禁用保护模式

# requirepass foobared #禁用保护模式后任何主机都可以连接
requirepass 123456     #设置密码

#重启
[root@ubuntu2 ~]$ systemctl restart redis-server.service

#远程主机
#下载redis数据库
[root@ubuntu2 ~]$ redis-cli -h 10.0.0.163  #远程连接
10.0.0.163:6379> info      #不输入密码无法获取数据
NOAUTH Authentication required.
10.0.0.163:6379> auth 123456   #输入密码
OK
10.0.0.163:6379> info
...

#主机端，仅支持10000个客户端连接
[root@ubuntu2 ~]$ cat /etc/redis/redis.conf | grep maxclients
# maxclients 10000

10.0.0.163:6379> config get maxclients
1) "maxclients"
2) "10000"

#修改最大连接用户数
[root@ubuntu2 ~]$ vim /usr/lib/systemd/system/redis-server.service 
...
[Service]
Type=notify
ExecStart=/usr/bin/redis-server /etc/redis/redis.conf --supervised systemd --daemonize no --maxclients 1
#重启
#查看配置，最大连接用户为1，其它远程客户端无法使用
127.0.0.1:6379> config get maxclients
1) "maxclients"
2) "1"

#也可在运行中修改，无法持久保存
127.0.0.1:6379> config set maxclients 2
OK
127.0.0.1:6379> config get maxclients
1) "maxclients"
2) "2"
#最大连接数是2，另外一个客户端可用
```

**客户端连接**

```bash
[root@ubuntu2 ~]$ redis-cli --help       #显示帮助
 -h 地址                                  #指定服务器ip，默认127.0.0.1
 -p 端口                                  #指定端口 默认6379
 -s                                      #指定socket文件
 -a                                      #指定连接密码
 -user                                   # 指定用户名
 -askpass                                #提示输入密码
 -u                                      #指定服务器url
 -r n 命令                               # 指定命令运行n次
 -r 3 -i 0.5 get key-1         #重复执行三次命令间隔0.5s
--key <file>                            #用于身份验证的私钥文件
--hotkeys                               #查找热键。仅在maxmemory-policy是*lfu时工作
--no-auth-warning                       #不输出认证时的警告信息

```

python语言连接（下载报错）

```bash

[root@ubuntu ~]# apt update;apt install python3-pip -y

[root@ubuntu ~]# pip install redis

[root@ubuntu ~]# cat test.py 
#!/bin/bash/python3
import redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)
r.set("test-key","hello")
value = r.get("test-key")
print(value)

```



##### 4、慢查询日志

4个阶段： 

1. client发送命令给redis

2. redis中命令排队

3. redis执行命令

4. redis将执行结果返回client

   

- 慢查询发生在redis执行命令时
- 客户端超时不一定慢查询，但慢查询是定户端超时的一个可能原因



```bash
#默认慢查询设置
[root@ubuntu2 ~]$ redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> config get *slow*
1) "slowlog-max-len"                    #默认保存最近128条慢查询
2) "128"
3) "slowlog-log-slower-than"            #10000微秒 执行时长超过10毫秒认为是慢查询
4) "10000"

#修改执行超时时长
127.0.0.1:6379> config set  slowlog-log-slower-than 1
OK
127.0.0.1:6379> config get *slow*
1) "slowlog-max-len"
2) "128"
3) "slowlog-log-slower-than"
4) "1"


127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379> set key1 123
OK
#有4条慢查询日志
127.0.0.1:6379> slowlog len
(integer) 4
#查看最近的一条
127.0.0.1:6379> slowlog get 1
1) 1) (integer) 4               #序号
   2) (integer) 1723448680      #执行命令的时间
   3) (integer) 1         
   4) 1) "slowlog"              #命令
      2) "len"
   5) "127.0.0.1:39716"
   6) ""
127.0.0.1:6379> slowlog reset
OK

```



##### 5、redis持久化

Redis 是基于内存型的 NoSQL，所有数据存放在内存中，机器重启断电，内存中数据会丢失。

redis持久化功能，可将内存中数据保存到磁盘上，开机重启，将磁盘数据加载到内存中

**RDB 持久化**

- RDB 持久化功能所生成的 RDB 文件是一个经过压缩的二进制文件，通过该文件可以还原生成该 RDB 文 件时数据库的状态。
- RDB 支持 save 和 bgsave 两种命令实现数据文件的持久化
- save命令使用主工作线程进行备份，不生成专门的子进程，速度快，不会堵塞
- save和bgsave执行过程中都会生成 temp-PID.rdb 的临时文件

优点：

- 只保存某个时间点的数据，恢复时直接加载到内存即可，适合用于灾备处理
- 可以自定义时间点保存多个版本的备份
- 备份速度快

确点：

- 不能实时保存数据，可能丢失上次备份到现在的数据。

- 数据量大时，fock（）进程会非常耗时，生成db也会花费很长时间

  ```bash
  [root@ubuntu2 ~]$ ls -lh /var/lib/redis/
  total 8.0K
  .....
  -rw-rw---- 1 redis redis  146 Aug 12 11:16 dump.rdb
  
  #默认持久化保存策略
  #3600S 内有一个KEY 发生变化就做持久化
  #300S 内有100个KEY 发生变化就做持久化
  #60S 内有 10000个KEY发生变化就做持久化
  [root@ubuntu ~]# redis-cli -c config get save
  1) "save"
  2) "3600 1 300 100 60 10000"
  
  #修改策略，无法用config 命令修改
  [root@ubuntu2 ~]$ vim /etc/redis/redis.conf 
  stop-writes-on-bgsave-error yes     #当快照失败是否仍允许客户端执行写操作
  rdbcompression yes                  #是否压缩数据
  rdbchecksum yes                     #是否对rdb文件进行校验
  dbfilename dump.rdb    #rdb 文件名
  dir /var/lib/redis     #rdb 文件默认路径
  
  # save 3600 1 300 100 60 10000
  save 30 3       #30S内有3个KEY发生变化就做持久化
  
  #重启
  [root@ubuntu2 ~]$ systemctl restart redis-server.service
  #查询
  [root@ubuntu2 ~]$ redis-cli -a 123456
  Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
  127.0.0.1:6379> config get save
  1) "save"
  2) "30 3"
  
  ```

  

**AOF持久化**

- AOF 和 RDB 都采有 COW 机制，AOF 可以指定不同的保存策略，默认为每秒 钟执行一次 fsync，按照操作的顺序地将变更命令追加至指定的 AOF 日志文件尾部
- 第一次启用 AOF 功能时，会做一次完全备份，后续将执行增量性备份，相当于完全数据备份+增量变化
- 如果同时启用 RDB 和 AOF 进行恢复时，默认 AOF 文件优先级高于 RDB 文件，

优点：

- 数据安全性相对较高，即每秒执行一次 fsync，在这种配置下，Redis 仍然可以 保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据
- 出现宕机现象，也不会破坏日志文件中已经存在的内容，Redis下一次启动之前，可以通过 redis-check-aof 工具来解决数 据一致性的问题
- AOF备份文件过大时，会自动重写，即使重写过程中发生停 机，现有的 AOF文件也不会丢失。而一旦新AOF文件创建完毕，Redis 就会从旧 AOF 文件切换到 新AOF文件，并开始对新 AOF 文件进行追加操作
- AOF文件的内容非常容易被人读懂

缺点

- 有重复操作也会被记录，文件一般比rdb大
- aof恢复大数据集的时候比rdb慢
- 如果 fsync 策略是appendfsync no, AOF保存到磁盘的速度甚至会可能会慢于RDB
- bug出现可能性高



```bash
[root@ubuntu2 ~]$ redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

#先在运行时开启 aof，保证 aof有全量数据及后需更新数据
127.0.0.1:6379> config set appendonly yes
OK
127.0.0.1:6379> config get appendonly
1) "appendonly"
2) "yes"
127.0.0.1:6379> 

#开启后产生了 appendonly 数据
[root@ubuntu2 ~]$ ls -l /var/lib/redis/
total 8
drwxr-x--- 2 redis redis 4096 Aug 12 09:15 appendonlydir
-rw-rw---- 1 redis redis  102 Aug 12 09:07 dump.rdb

[root@ubuntu2 ~]$ ls -l /var/lib/redis/appendonlydir/
total 8
-rw-rw---- 1 redis redis 142 Aug 12 09:15 appendonly.aof.1.base.rdb
-rw-r----- 1 redis redis   0 Aug 12 09:15 appendonly.aof.1.incr.aof
-rw-r----- 1 redis redis  88 Aug 12 09:15 appendonly.aof.manifest
#数据读写
[root@ubuntu2 ~]$ redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set test-1 aa
OK
127.0.0.1:6379> set test-2 bb
OK
127.0.0.1:6379> set test-3 c
OK
127.0.0.1:6379> keys test*
1) "test-1"
2) "test-4"
3) "test-3"
4) "test-2"
127.0.0.1:6379> DBSIZE
(integer) 5
127.0.0.1:6379>

[root@ubuntu2 ~]$ ls -l /var/lib/redis/appendonlydir/
total 12
-rw-rw---- 1 redis redis 142 Aug 12 09:15 appendonly.aof.1.base.rdb   #全量没更新
-rw-r----- 1 redis redis 121 Aug 12 09:18 appendonly.aof.1.incr.aof   #增量更新了 

-rw-r----- 1 redis redis  88 Aug 12 09:15 appendonly.aof.manifest

#都是写数据的过程
[root@ubuntu2 ~]$ cat /var/licat /var/lib/redis/appendonlydir/appendonly.aof.1.incr.aof
*2
$6
SELECT
$1
0
*3
$3
set
$6
test-1
$2
aa
*3
$3
set
$6
test-2
$2
bb
*3
$3
set
$6
test-3
$1
c
#配置文件中开启aof
[root@ubuntu2 ~]$ vim /etc/redis/redis.conf

appendonly yes
#重启后数据都在
[root@ubuntu2 ~]$ systemctl restart redis
[root@ubuntu2 ~]$ redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> DBSIZE
(integer) 5
127.0.0.1:6379> key test*

```

```bash
#利用aof还原误操作的数据
[root@ubuntu2 ~]$ redis-cli -a 123456
127.0.0.1:6379> DBSIZE
(integer) 5
127.0.0.1:6379> key test*
(error) ERR unknown command 'key', with args beginning with: 'test*' 
127.0.0.1:6379> keys test*
1) "test-2"
2) "test-4"
3) "test-1"
4) "test-3"
127.0.0.1:6379> DBSIZE
(integer) 5
127.0.0.1:6379> keys test*
1) "test-2"
2) "test-4"
3) "test-1"
4) "test-3"
127.0.0.1:6379> flushdb #删除数据
OK
127.0.0.1:6379> set test-3 cc #更改test-3 test-4
OK
127.0.0.1:6379> set test-4 dd
OK
127.0.0.1:6379> dbsize
(integer) 2
127.0.0.1:6379> keys *
1) "test-3"
2) "test-4"
127.0.0.1:6379> exit

#操作过程都在此文件中，将此文件中的误操作事务去掉，再重启服务即可
[root@ubuntu2 ~]$ vim /var/lib/redis/appendonlydir/appendonly.aof.1.incr.aof 

*2
$6
SELECT
$1
0
*3
$3
set
$6
test-1
$2
aa
*3
$3
set
$6
test-2
$2
bb
*3
$3
set
$6
test-3
$1
c
*2
$6
SELECT
$1
0
*1 #此此行开始
$7
flushdb #此行结束，都删掉
*3
$3
set
$6
test-3
$2
cc
*3
$3
[root@ubuntu2 ~]$ systemctl restart redis
#数据已还原，清除之后更改的数据没变
[root@ubuntu2 ~]$ redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> keys test*
1) "test-2"
2) "test-1"
3) "test-3"
4) "test-4"
127.0.0.1:6379> dbsize
(integer) 5
127.0.0.1:6379> 

```

```bash
#AOF Rewrite重写

#进行多次set操作，最终值是5555
127.0.0.1:6379> set test-5 5
OK
127.0.0.1:6379> set test-5 55
OK
127.0.0.1:6379> set test-5 555
OK
127.0.0.1:6379> set test-5 5555
OK
127.0.0.1:6379> keys test*
1) "test-4"
2) "test-3"
3) "test-5"
4) "test-1"
5) "test-2"
127.0.0.1:6379> get test-5
"5555"

#aof中会记录多次操作，前几次是无用的
[root@ubuntu2 ~]$ cat /var/lib/redis/appendonlydir/appendonly.aof.1.incr.aof 
......
set
$6
test-5
$1
5
*3
$3
set
$6
test-5
$2
55
*3
$3
set
$6
test-5
$3
555
*3
$3
set
$6
test-5
$4
5555

[root@ubuntu2 ~]$ ls -l /var/lib/redis/appendonlydir/
total 12
-rw-rw---- 1 redis redis 142 Aug 12 09:15 appendonly.aof.1.base.rdb
-rw-r----- 1 redis redis 367 Aug 12 11:18 appendonly.aof.1.incr.aof
-rw-r----- 1 redis redis  88 Aug 12 09:15 appendonly.aof.manifest
#开始整理aof
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started

#整理完查看aof，生成新的aof文件，aof.1.incr文件合并到rdb中了
[root@ubuntu2 ~]$ ls -l /var/lib/redis/appendonlydir/
total 8
-rw-rw---- 1 redis redis 157 Aug 12 11:20 appendonly.aof.2.base.rdb
-rw-r----- 1 redis redis   0 Aug 12 11:20 appendonly.aof.2.incr.aof
-rw-r----- 1 redis redis  88 Aug 12 11:20 appendonly.aof.manifest
[root@ubuntu2 ~]$ cat /var/lib/redis/appendonlydir/appendonly.aof.2.incr.aof 
```

如果缓存能承受几分钟的数据丢失，一般启用rdb即可，

数据一点不能丢失可以同时开启rdb和aof ，不建议只开启aof

##### 6、redis常见命令和数据类型

- info [selection...]     显示服务器状态信息
- select N                    切换数据库，默认有16个数据库，编号 0-15，在Redis cluster 模式下不支持多 个数据库
- keys pattern            查看当前库中的所有的 key，可以使用通配符
- bgsave                      在后台执行 rdb 持久化
- save                          阻塞执行 rdb 持久化
- dbsize                       统计当前数据库中有多少个KEY flushdb 清空当前库
- flushall                      清空所有库
- shutdown                 关闭服务

```bash
#默认输出所有信息
127.0.0.1:6379> info 
...
#输出指定sestion
127.0.0.1:6379> info clients cluster
# Clients
connected_clients:1
...
# Cluster
cluster_enabled:0

#切换数据库
root@ubuntu2 ~]$ cat /etc/redis/redis.conf | grep databases
databases 16  #默认0-15 16个数据库 redis cluster模式下不支持多个数据库
#默认是0号数据库
root@ubuntu2 ~]$ redis-cli -a 123456
127.0.0.1:6379> DBSIZE  #查看数据库中有多少个key
(integer) 6
127.0.0.1:6379> select 1 #切换到1号数据库
OK
127.0.0.1:6379[1]> DBSIZE 
(integer) 0

#每个库的数据无法和其它库共享
127.0.0.1:6379[1]> set db1-key1 abc
OK
127.0.0.1:6379[1]> set key1 1
OK
127.0.0.1:6379[1]> keys * #列出库中所有key *是通配符 #数据量大会阻塞很长时间
1) "key1"
2) "db1-key1"
127.0.0.1:6379[1]> select 2
OK
127.0.0.1:6379[2]> keys *
(empty array)
#key * 命令时间复杂度为O(N)，慎用，如果当前库中数据量较大，此命令会执行很长时间，会阻塞其它命令，导致客户端其它命令读写操作失败

#清空数据库
127.0.0.1:6379[1]> flushdb
OK
127.0.0.1:6379[1]> DBSIZE
(integer) 0

#flushdb，flushall 命令和 appendonly 冲突，无法重放 （重启报错）
[root@ubuntu ~]# cat /var/log/redis/redis-server.log
3954:M 07 Jul 2024 23:27:52.347 # Unknown command 'flushdb' reading the append 
only file appendonly.aof.2.incr.aof


127.0.0.1:6379[1]> flushall #清空所有库数据
OK

#flushdb 和 flushall 都会清空数据，且和 appendonly 冲突，可以在配置文件中禁用危险命令
#可以用 rename-command 将要禁用的命令定义成空或一个超长的字符串
[oot@ubuntu ~]# cat /etc/redis/redis.conf
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command flushdb ""
rename-command flushall ""
#重启（报错）

#查看key是否存在
127.0.0.1:6379> exists key-1
(integer) 0
127.0.0.1:6379> exists test-1
(integer) 1

#删除key
127.0.0.1:6379> del key1

#查询 key 的生命周期，-1 表示永不过期，-2表示key不存在
127.0.0.1:6379> ttl key-1
(integer) -2
127.0.0.1:6379> ttl test-1
(integer) -1

#设置生命周期
127.0.0.1:6379> expire test-1  15 #设置15s生命周期
(integer) 1
127.0.0.1:6379> ttl test-1
(integer) 5                       #还有5s
127.0.0.1:6379> ttl test-1
(integer) 3
#移除TTL，变成永不过期
#127.0.0.1:6379> PERSIST test-1
127.0.0.1:6379> ttl test-1
(integer) -2                       #key不存在


#通配符
127.0.0.1:6379> keys ?est??
1) "test-b"
2) "test-a"
127.0.0.1:6379> keys key*
1) "key-1"
2) "key-2"

#移动key到另一个db
127.0.0.1:6379> move test-2 2
(integer) 1
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> keys *
1) "test-2"

#获取随机key
127.0.0.1:6379> keys *
1) "test-4"
2) "test-3"
3) "test-5"
127.0.0.1:6379> randomkey
"test-4"
127.0.0.1:6379> randomkey
"test-3"
#重命名 key-1不存在报错，key-2存在被覆盖
127.0.0.1:6379> rename key-1 key-2
OK
#目标 key 存在，不会被覆盖
127.0.0.1:6379> RENAMENX key-2 key-123
(integer) 0

#类型
127.0.0.1:6379> type key-2
string
127.0.0.1:6379> hset hash-1 a 123 b 456
(integer) 2
127.0.0.1:6379> type hash-1
hash

```

##### 7、Redis 中的数据类型

Redis 是一个 Key-Value 类型的键值对缓存数据库，

value可支持多种数据类型：string、hash、list、set（集合）、zset（有序集合）、Bitmaps（位图）、HyperLogLogs（超日志）、Geospatial（地理空间）、Pub/Sub：（发布/订阅）、Streams（流）、Modules（模块）



##### 8、string类型

string 是 redis 最基本的数据类型，string 类型是二进制安全的，value 可以包含任何数据，比如图片或 序列化的对象，string 类型的值最大能存储 512MB

```bash
#追加 key存在时追加，不存在时创建新的key
127.0.0.1:6379> get test-5
"5555"
127.0.0.1:6379> append test-5 123
(integer) 7
127.0.0.1:6379> get test-5
"5555123"

#自减 
127.0.0.1:6379> set key-1 123abc
OK
127.0.0.1:6379> decr key-1  #必须是整数，负数
(error) ERR value is not an integer or out of range
127.0.0.1:6379> set key-1 123
OK
127.0.0.1:6379> decr key-1   #默认是自减1
(integer) 122
127.0.0.1:6379> decrby key-1 3 #设置自减3
(integer) 119

#自增 key不存在时，自增后key为1
127.0.0.1:6379> incr key-1 #自增默认为1
(integer) 11
127.0.0.1:6379> incrby  key-1 10 #自定义增量为10
(integer) 21
127.0.0.1:6379> 

#取子串
127.0.0.1:6379> set key-1 abcdefg
OK
127.0.0.1:6379> getrange key-1  0 3 #从0开始
"abcd"
127.0.0.1:6379> getrange key-1  0 11 #越界
"abcdefg"
127.0.0.1:6379> getrange key-1  0 -2 #负数是从后向前数
"abcdef"

#设置新值返回旧值
127.0.0.1:6379> get key-1
"abcdefg"
127.0.0.1:6379> getset key-1 10 #如果key-1不存在设置新key，返回空
"abcdefg"

127.0.0.1:6379> mset key-11 11 key-22 22 #批量写（key-11 已存在批量写失败）
OK
127.0.0.1:6379> mget key-11 key-22 key-33 #批量读
1) "11"
2) "22"
3) (nil)

#设置key-2 ttl为20s
127.0.0.1:6379> set key-2 10 ex 20
OK
127.0.0.1:6379> ttl key-2
(integer) 0
127.0.0.1:6379> ttl key-2
(integer) -2

127.0.0.1:6379> set key-3333 3333 EX 60 NX #设置 key，并指定 TTL 60S，且在 key3333不存在时才创建

127.0.0.1:6379> setex key-5 60 55                    #设置KEY 并指定TTL
OK
127.0.0.1:6379> setex key-5 90 555                    #覆盖写改变了值和ttl
OK

#写，如果有，写失败
127.0.0.1:6379> set key-2 20
OK
127.0.0.1:6379> setnx key-2 30
(integer) 0
127.0.0.1:6379> setnx key-3 30 
(integer) 1

127.0.0.1:6379> get key-5
"abcdefg"
127.0.0.1:6379> setrange key-5 3 123456 #从指定位置开始覆盖写，下标从0开始
(integer) 9
127.0.0.1:6379> get key-5
"abc123456"

#越界用\x00补充
127.0.0.1:6379> set  key-5 abc
OK
127.0.0.1:6379> setrange key-5 6 123
(integer) 9
127.0.0.1:6379> get key-5
"abc\x00\x00\x00123"
127.0.0.1:6379> strlen key-5 #内容长度
(integer) 9

```

##### 9、列表-list

管道：单向读写              列表：双向读写