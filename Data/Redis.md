# 引言

## 介绍

以Key-Value存储数据的高性能NoSQL内存数据库

* 特性
  * 高性能Key-Value数据库
  
  * 支持不同层都的数据持久化
  
  * Value支持多种数据结构
  
  * 支持主从复制 ( Master-Slave ) 的备份模式
  
  * 单线程, 所有操作都是原子性的, 也支持多操作的事务
  
    > 这里的事务其实是批处理, 没有回滚操作.
  
  * 发布/订阅模式
  
  * 支持 Lua 脚本
  
  * LRU eviction 算法
  
  * 哨兵与集群
* Value可以是以下数据结构
  * 字符串String
  * 哈希Hash
  * 列表List
  * 集合Set
  * 有序集合Sorted Set  with range queries 
  *  bitmaps 
  *  hyperloglogs 
  *  geospatial indexes  with radius queries 
  *  streams 

## 安装

`Ubuntu`下,

1.  安装

   ```shell
   sudo apt install redis
   ```

2. 启动服务

   ```shell
   redis-server &
   ```

   或者

   ```shell
   systemctl start redis-server
   ```

3. 启动客户端

   ```shell
   redis-cli
   ```

> 问题: 
>
> redis-server.service: Can't open PID file /var/run/redis/redis-server.pid (yet?) after start: No such file or directory
>
> 排查:
>
> 查看日志` /var/log/redis/redis-server.log `, 发现端口被占用
>
> `ss`查看端口, 发现监听了其ip6的6379端口, 又去监听ip4的6379端口, 原因明了.
>
> 解决:
>
> 配置文件`/etc/redis/redis.conf`中
>
> ```property
> bind 127.0.0.1 ::1
> ```
>
> 去掉`::1`
>
> > 参考 https://askubuntu.com/questions/967763/redis-server-service-failed-with-result-timeout-more-errors-listed-inside 

# 命令

## 基本操作

* `SET` 设置键值对

* `Get` 获取值

* `DEL` to delete a given key and associated value 

* `SETNX` sets a key only if it does not already exist 

* `INCR` to atomically increment a number stored at a given key 

* `EXPIRE` 设置键值的过期时间, 默认永不过期(无过期时间)

* `TTL` 返回键值存在的剩余时间. 

  >`-2`表示键已过期, `-1`表示永不过期
  
* `keys` 查看匹配模式的所有key

* `flushdb`删除当前数据库中的所有Key

* `flushall`删除所有数据库中的key

  

















> 参考
>
> *  https://redis.io/ 