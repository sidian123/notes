# Getting Start

## 介绍

* Elasticsearch是一个分布式文档数据库
* 常用作[全文检索](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95/1140318?fr=aladdin), 和日志存储
* 提供Rest API的方式访问数据库
* 若需要对中文进行全文索引, 需安装中文分词插件(原因见全文索引原理), 然后建立Type时, 设置需要全文索引的字段使用中文分词器.

## 安装 & 运行

```shell
docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
```

> 上述运行单实例Elasticsearch, 生产环境需配置多个实例

接着, 运行`curl localhost:9200`命令, 检查程序是否启动成功

若要可视化查看Elasticsearch, 可安装Kibana, 见下.

# 基本概念

## 全文索引(倒排索引)

* 原理

  先定义一个词库，然后在文章中查找每个**词条(term)**出现的频率和位置，把这样的频率和位置信息按照词库的顺序归纳，这样就相当于对文件建立了一个以词库为目录的索引，这样查找某个词的时候就能很快的定位到该词出现的位置。

* 问题

  英文可通过空格将词条(term)分离出来, 并建立索引. 但是中文不好分词, 若每个字都分词, 那索引项会巨多, 全文搜索将毫无效率; 若人工分词, 维护成功将很高. 

* 解决方法

  二元法, 词库法, 正向最大匹配+逆向最大匹配, .... ???

> 参考[全文索引](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95/1140318?fr=aladdin)

## Node & Cluster

Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

单个 Elastic 实例称为一个节点（node）; 一组节点构成一个集群（cluster）。

## Index

Elastic 会索引所有文档的所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。

可类比为数据库. 每个Index名字必须小写?

## Document

Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。

同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。

## Type

Type是对Document某个字段不同的取值而做出的虚拟分组, 可类比为表. 不同的 Type 应该有相似的结构（schema）

根据[规划](https://www.elastic.co/blog/index-type-parent-child-join-now-future-in-elasticsearch)，Elastic 6.x 版只允许每个 Index 包含一个 Type，7.x 版将会彻底移除 Type ???

## Document metadata

文档元数据为_index, _type, _id, 这三者可以唯一表示一个文档，_index表示文档在哪存放，_type表示文档的对象类别，_id为文档的唯一标识。

对指定`_id`的文档修改或新增, `_version`会自增

## Fields

每个Document都类似一个JSON结构，它包含了许多字段，每个字段都有其对应的值，多个字段组成了一个 Document，可以类比关系型数据库数据表中的字段。

## 关系数据库与Elasticsearch对比

![img](.Elasticsearch/9419034-4f8eb4926bc326de.png)

# Rest API

## 插入数据

向`/Index/Type`发送PUT请求

```
http://localhost:9200/accounts/person
{
  "_index": "accounts",
  "_type": "person",
  "_id": "23",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 4,
  "_primary_term": 1
}
```

上述自动生成文档`_id`, 也可手动指定

```
http://localhost:9200/accounts/person/24
{
  "_index": "accounts",
  "_type": "person",
  "_id": "24",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 7,
  "_primary_term": 1
}
```

# Kibana

* 介绍

  Kibana 是一个开源的分析和可视化平台，旨在与 Elasticsearch 合作。Kibana 提供搜索、查看和与存储在 Elasticsearch 索引中的数据进行交互的功能。开发者或运维人员可以轻松地执行高级数据分析，并在各种图表、表格和地图中可视化数据。

* 运行

  ```shell
  docker run -d --name kibana -p 5601:5601 kibana:7.8.0
  ```

  打开http://localhost:5601可访问到kibana
  
  > 注意, 还需要将elasticsearch和kibana加入到同一自定义bridge网络中, 否则容器间无法通信. (见[docker网络知识](https://sidian.live/article/?id=268#head-15-0-0-0-0-0))
  
* 参考

  * [Configure Kibana](https://www.elastic.co/guide/en/kibana/current/settings.html) kibana的配置
  * [Install Kibana with Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)

# 参考

* [全文搜索引擎 Elasticsearch 入门教程 阮一峰](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)
* [Elasticsearch简介与实战](https://www.jianshu.com/p/d48c32423789)

