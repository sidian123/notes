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

### APT安装

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

   > 端口监听端口6379

3. 启动客户端

   ```shell
   redis-cli
   ```

### 源码安装

1. 下载 & 编译

   ```shell
   $ wget http://download.redis.io/releases/redis-5.0.5.tar.gz
   $ tar xzf redis-5.0.5.tar.gz
   $ cd redis-5.0.5
   $ make
   ```

2. 此时可执行文件位于`src/`目录

   ```shell
   src/redis-server
   ```

   可指定配置文件

   ```shell
   ./redis-server /path/to/redis.conf
   ```

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
  
* `keys` 查看匹配模式的所有key, 支持通配符

* `flushdb`删除当前数据库中的所有Key

* `flushall`删除所有数据库中的key

* `auth`认证

* `config set|get`获取或设置配置

* `client list`查看所有连接

* `info`查看Redis所有信息

## 配置

* 密码: 远程访问Redis将不被运行, 如果设置了密码, 则可以. 需在配置文件中添加

  ```config
  requirepass 123456
  ```

* `bind` 配置绑定的ip地址. 若要外网能够访问, 则配置为

  ```property
  bind 0.0.0.0
  ```

# 待看

[浅谈redis的单点模式、主从模式、哨兵模式和集群模式](https://blog.csdn.net/m0_38084879/article/details/100542327)

# 坑

## 端口占用

* 错误

  ```
  redis-server.service: Can't open PID file /var/run/redis/redis-server.pid (yet?) after start: No such file or directory
  ```

* 排查

  查看日志` /var/log/redis/redis-server.log `, 发现端口被占用

  `ss`查看端口, 发现监听了其ip6的6379端口, 又去监听ip4的6379端口, 原因明了.

* 解决

  配置文件`/etc/redis/redis.conf`中

    ```property
    bind 127.0.0.1 ::1
    ```

	去掉`::1 `

> 其实还未解决上述问题, 但却解决了阻止ngnix运行的问题, 也就不必再深究了.

> 参考 https://askubuntu.com/questions/967763/redis-server-service-failed-with-result-timeout-more-errors-listed-inside 

## Redis不允许备份

```
MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk
```

这是因为Redis fork子进程来备份的, OS认为该行为将消耗大量内存而拒绝. 更改内存策略即可:

1. `/etc/sysctl.conf`中添加

   ```conf
   vm.overcommit_memory=1
   ```

2. 重新读取系统参数

   ```shell
   sysctl -p /etc/sysctl.conf
   ```

> 详细解析见https://stackoverflow.com/a/49839193

# 参考

* https://redis.io/ 