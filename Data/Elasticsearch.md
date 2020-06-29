# Getting Start

* 介绍

  * Elasticsearch是一个分布式文档数据库
  * 常用作[全文检索](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95/1140318?fr=aladdin), 和日志存储
  * 提供Rest API的方式访问数据库
  * 若需要对中文进行全文索引, 需安装中文分词插件(原因见全文索引原理), 然后建立Type时, 设置需要全文索引的字段使用中文分词器.
  
* 安装 & 运行

  ```shell
  docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
  ```
  
  > 上述运行单实例Elasticsearch, 生产环境需配置多个实例
  
  接着, 运行`curl localhost:9200`命令, 检查程序是否启动成功
  
* 基本概念

  * Node & Cluster

    Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

    单个 Elastic 实例称为一个节点（node）; 一组节点构成一个集群（cluster）。

  * Index

    Elastic 会索引所有文档的所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。

    可类比为数据库. 每个Index名字必须小写?

  * Document

    Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。

    同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。

  * Type

    Type是对Document某个字段不同的取值而做出的虚拟分组, 可类比为表. 不同的 Type 应该有相似的结构（schema）

    根据[规划](https://www.elastic.co/blog/index-type-parent-child-join-now-future-in-elasticsearch)，Elastic 6.x 版只允许每个 Index 包含一个 Type，7.x 版将会彻底移除 Type ???

# Kibana

* 介绍

  Kibana is an open source analytics and visualization platform designed  to work with Elasticsearch. You use Kibana to search, view, and interact with data stored in Elasticsearch indices. You can easily perform  advanced data analysis and visualize your data in a variety of charts,  tables, and maps.

* 运行

  ```shell
  docker run -d --name kibana -p 5601:5601 kibana:7.8.0
  ```

  打开http://localhost:5601可访问到kibana
  
  > 注意, 还需要将elasticsearch和kibana加入到同一自定义bridge网络中, 否则荣期间无法通信. (见[docker网络知识](https://sidian.live/article/?id=268#head-15-0-0-0-0-0))

# 参考

* [全文搜索引擎 Elasticsearch 入门教程 阮一峰](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)