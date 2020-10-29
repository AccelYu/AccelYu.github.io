---
layout: post
title: Redis（三），架构
categories: [数据库]
---

Redis是单线程的，使用IO多路复用，所以它是线程安全的

<!-- more -->

## 架构
### 单机版
1. 优点：简单

2. 缺点：内存容量有限、处理能力有限、无法高可用

### 主从复制
1. 优点：
   1. 数据冗余，实现数据的热备份
   2. 故障恢复，避免单点故障带来的服务不可用
   3. 读写分离，负载均衡。主节点负载读写，从节点负责读，提高服务器并发量
   4. 高可用基础，是哨兵机制和集群实现的基础

2. 缺点：没有缓解（主节点的）写压力

3. 容灾处理  
当master出现故障，需手动将slave中的一个提升为master，剩下的slave挂至新的master上

4. 实现方式：
    ```
    # 原理
    1、slave发送请求建立socket连接，master验证密码，通过后保存slave的ip和监听端口
    slave保存master返回的runid
    
    2、slave向master发送PSYNC请求（Redis2.8版本以上）->PSYNC runid offset，runid为master的id，
    offset为偏移量（master和slave双方都会各自维持一个offset。
    Master成功发送N个字节的命令后会将Master的offset加上N，Slave在接收到N个字节命令后同样会将Slave的offset增加N）
    第一次连接会发送->PSYNC ? -1，然后进行完整重同步
    
    3、master接收到PSYNC命名后，开始执行BGSAVE并使用复制积压缓冲区记录此后执行的所有写命令
    
    4、slave写入rdb后，将复制积压缓冲区内数据也写入，完成数据同步
    
    5、repl-backlog-size复制积压缓冲区所有slave共用。
    当slave断线重连后发送的runid不对或复制积压缓冲区中需要的数据已删除则进行完整重同步，
    否则进行部分重同步，master比对slave发来的offset，然后返回slave缺失的数据
    
    6、同步完成后，开始master命令传播。slave默认以每秒一次的频率向master发送心跳检测REPLCONF ACK <replication_offset>
    作用是检测主从服务器的网络连接状态
    辅助实现min-slaves配置（min-slaves-to-write从服务器的数量少于，min-slaves-max-lag从服务器的延迟高于，master将拒绝执行写命令）
    检测命令丢失（master补发缺失数据的过程类似于部分重同步）
    ```
    ```
    # 在redis.conf文件中配置可永久生效，命令行设置的话重启会失效
    slaveof <masterip> <masterport>
    # 在5.0版本中使用了replicaof代替了slaveof
    replicaof <masterip> <masterport>
    # 从节点设置为只读
    slave-read-only yes
    # 主节点的密码
    masterauth pwd
    # 设置复制积压缓冲区大小，默认1mb
    repl-backlog-size 64mb
    ```

### 哨兵（Redis2.8）
1. 优点
   1. 主从可以自动切换，系统更健壮，可用性更高
   2. 监控了各个节点

2. 缺点：
   1. 主从切换需要时间，会丢失数据
   2. 没有解决master写的压力
   
3. 实现方式：
    ```
    # 原理
    1、sentinel定时任务
    每1秒每个sentinel对其他sentinel和redis节点执行ping操作
    每2秒每个sentinel通过 Master节点的channel交换信息
    每10秒每个sentinel会对master和slave执行info命令,用于发现slave节点和确认主从关系
    
    2、主管下线
    在给定时间内，节点未做出回应，sentinel标记该节点主观下线sdown
    
    3、客观下线
    主管下线后该sentinel向其它sentinel发送is-master-down-by-addr
    多个sentinel对同一个节点做出sdown，则标记该节点客观下线odown，然后开启故障转移failover
    当master被认定为客观下线时，才会发生故障迁移
    
    4、如果 Master处于odown，则投票自动选出新的master，所有slave指向新master，并修改所有的节点配置文件
    ```
    ```
    # 在sentinel.conf中配置
    #设置 主名称 ip地址 端口号 开启选举的哨兵数
    sentinel monitor mymaster 192.168.194.131 6381 quorum
    #主观下线时长
    sentinel down-after-milliseconds mymaster 3000
    #每次最多可以有1个从同步主。数量不宜过多，否则导致大量slave不可以
    sentinel parallel-syncs mymaster 1
    #若哨兵在配置值内未能完成故障转移操作，则任务本次故障转移失败
    sentinel failover-timeout mymaster 18000
    # 主节点密码
    sentinel auth-pass mymaster 123456
    ```

### 集群（proxy型）
1. 优点：
   1. 多种hash算法：MD5、CRC16、CRC32、CRC32a、hsieh、murmur、Jenkins
   2. 支持失败节点自动删除
   3. 后端Sharding分片逻辑对业务透明，业务方的读写方式和操作单个Redis一致

2. 缺点：增加了新的 proxy，需要维护其高可用

### 集群（直连型redis3.0）
1. 优点：
   1. 无中心架构（不存在哪个节点影响性能瓶颈），少了 proxy 层。
   2. 数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布。
   3. 可扩展性，可线性扩展到 1000 个节点，节点可动态添加或删除。
   4. 高可用性，部分节点不可用时，集群仍可用。通过增加 Slave 做备份数据副本
   5. 实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave到 Master 的角色提升。

2. 缺点：
   1. 资源隔离性较差，容易出现相互影响的情况
   2. 数据通过异步复制，不保证数据的强一致性
   
3. 实现方式：
```
# 原理
1、分片
hash槽分配，redis会大致均等的将哈希槽映射到不同的节点
计算键所在的哈希槽，HASH_SLOT = CRC16(key) % 16384，将键写入拥有这个槽的节点

2、哈希槽
使用哈希槽而不是一致性哈希，
槽位数量是2^14而不是2^16-1，

3、扩容和缩容（增删主节点）
hash槽重分配，其它节点从自己所有的哈希槽中抽取一部分给新的节点完成扩容
需删除节点将自己的哈希槽分给其它节点完成缩容

4、故障自动转移，参考主从复制
```
```
# redis.conf文件中配置
# 开启集群
cluster-enabled yes
# 集群的配置，首次启动自动生成
cluster-config-file  nodes_ip.conf
# 请求超时单位毫秒
cluster-node-timeout  5000
```
```
# docker运行容器
docker run -d -p 7001:6379 -v /home/redis.conf:/data/redis.conf --name redis01 redis:3.2 redis-server /data/redis.conf
# 查看所有容器ip
docker inspect -f='{{.NetworkSettings.IPAddress}}' $(docker ps -a -q)
```
```
# 命令行（redis5之前）
# 原生搭建
redis-cli -c -h ip -p port
cluster meet new_ip new_port

# 分配槽
redis-cli -h ip -p port cluster addslots {0..5461}

# 添加从节点
redis-cli -h ip -p port cluster replicate master_id
```
```
# 命令行（redis5及以后）
# 使用客户端搭建

# 创建集群，至少三个主节点，cluster-replicas设置主从个数比例
redis-cli --cluster create ip1:port1 ip2:port2 ip3:port3 --cluster-replicas 1

# 额外添加主节点
redis-cli --cluster add-node new_ip:new_port exist_ip:exist_port

# 额外添加从节点
redis-cli --cluster add-node new_ip:new_port --cluster-slave --cluster-master-id id

# 删除节点（推荐先删子节点，再删主节点）
redis-cli --cluster del-node id

# 重分配槽，数据也会被迁移
redis-cli --cluster reshare exist_ip:exist_port
redis-cli --cluster reshare exist_ip:exist_port --cluster-from id1 --cluster-to id2 --cluster-slots num
```