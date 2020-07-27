# 开始

## 介绍

* Elasticsearch是一个分布式文档数据库
* 常用作[全文检索](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95/1140318?fr=aladdin), 和日志存储
* 提供Rest API的方式访问数据库

* 对中文进行全文索引时, 最好配置中文分词插件(原因见原理)

## 基本概念

* 文档 Document

  被索引到的基本单元, 可以代表一个文档, 文档或商品等, 以JSON对象描述.

* 索引 Index

  是一种倒排索引, 直接建立**词**与文档的映射关系. 定义了要索引的字段, 和分词方式.

* 类型 Type

  文档的类型. 类型上也可以配置使用的分词器.

--------

* 元数据

  * `_index, _type, _id`

    文档元数据为, 这三者可以唯一表示一个文档，_index表示文档在哪存放，_type表示文档的对象类别，_id为文档的唯一标识。

  * `_version`

    文档版本. 对指定`_id`的文档修改或新增时, `_version`会自增

  * `_score`

    查询时对文档打的分数, 分数越高, 排序越靠前

* Node & Cluster

  Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

  单个 Elastic 实例称为一个节点（node）; 一组节点构成一个集群（cluster）。

-----

* 关系

  * 一个索引内, 可存在多个类型, 多个文档
  *  一个文档只能属于一个类型. 

  * 类型最好有相似的结构. 若结构极大不同, 则因放入不同索引内.

* vs. 关系数据库

  ![img](.Elasticsearch/9419034-4f8eb4926bc326de.png)

## 原理

### 倒排索引

* 索引过程

  维护一个分词库, 根据分词库对*文档*进行分词. 以分词库为目录, 建立一个索引, 记录分词与文档的映射关系.

* 查询过程

  在索引表中, 以O(log2n)的时间复杂度找到要查询的分词, 接着直接得到含有分词的文档位置.

### 分词器

#### 类型

* 默认分词 standard

  英文按照空格分词, 中文每个都分词. 这样分出的词无法携带语义, 查找出来的内容极有可能不是我们想要的.

* IK分词器

  一个开源的, 第三方分词器, 效果还行. 提供了两种分词器, 

  * `ik_smart` 做最粗粒度的拆分
  * `ik_max_word` 做最细粒度的拆分

* 其他

  smartCN,HanLP

#### IK分词器使用

安装

```java
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip
```

创建索引时, 为中文字段设置分词器, 如

```bash
$ curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}'
```

## 安装 & 运行

```shell
docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
```

> 上述运行单实例Elasticsearch, 生产环境需配置多个实例

接着, 运行`curl localhost:9200`命令, 检查程序是否启动成功

> 若要可视化查看Elasticsearch, 可安装Kibana, 见下.

# 结构化操作

> 新建文档时, 若索引和类型不存在, 会自动创建

提供了Rest风格的API供访问.

## 索引

* 新建索引

    ```bash
    $ curl -X PUT 'localhost:9200/weather'
    ```

    新建的同时配置类型和字段

    ```bash
    $ curl -X PUT 'localhost:9200/accounts' -d '
    {
      "mappings": {
        "person": {
          "properties": {
            "user": {
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            },
            "title": {
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            },
            "desc": {
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            }
          }
        }
      }
    }'
    ```
    
* 删除索引

    ```bash
    $ curl -X DELETE 'localhost:9200/weather'
    ```

* 列出所有索引

    ```bash
    $ curl -X GET 'http://localhost:9200/_cat/indices?v'
    ```
    
* 查看索引信息

    ```python
    curl -X GET "http://127.0.0.1:9200/productindex/_mapping?pretty" 
    ```

    > `mappings`包含所有类型信息

## 类型

* 所有的类型

  ```bash
  $ curl 'localhost:9200/_mapping?pretty=true'
  ```
  
* 为索引添加类型

  ```shell
  curl -XPOST "http://127.0.0.1:9200/productindex/product/_mapping?pretty" -d ' 
  {
      "product": {
              "properties": {
                  "title": {
                      "type": "string",
                      "store": "yes"
                  },
                  "description": {
                      "type": "string",
                      "analyzer": "standard"
                  },
                  "price": {
                      "type": "double"
                  },
                  "onSale": {
                      "type": "boolean"
                  },
                  "type": {
                      "type": "integer"
                  },
                  "createDate": {
                      "type": "date"
                  }
              }
          }
    }
  }'
  ```

* 新增一个字段

  ```shell
  curl -X POST "http://127.0.0.1:9200/productindex/product/_mapping -d '
  
  {
       "product": {
                  "properties": {
                       "amount":{
                          "type":"integer"
                     }
                  }
      }
  }'
  ```

* 修改字段? 不被允许的, 否则必将导致所有数据都要重新索引一遍

## 文档

* 新增文档

  指定id添加

  ```bash
  $ curl -X PUT 'localhost:9200/accounts/person/1' -d '
  {
    "user": "张三",
    "title": "工程师",
    "desc": "数据库管理"
  }' 
  ```

  自动生成id

  ```bash
  $ curl -X POST 'localhost:9200/accounts/person' -d '
  {
    "user": "李四",
    "title": "工程师",
    "desc": "系统管理"
  }'
  ```

* 查看记录

  ```bash
  $ curl 'localhost:9200/accounts/person/1?pretty=true'
  ```

  > `pretty`参数表示以更易读的方式返回的JSON

* 删除记录

  ```bash
  $ curl -X DELETE 'localhost:9200/accounts/person/1'
  ```

* 更新记录

  ```bash
  $ curl -X PUT 'localhost:9200/accounts/person/1' -d '
  {
      "user" : "张三",
      "title" : "工程师",
      "desc" : "数据库管理，软件开发"
  }' 
  
  {
    "_index":"accounts",
    "_type":"person",
    "_id":"1",
    "_version":2,
    "result":"updated",
    "_shards":{"total":2,"successful":1,"failed":0},
    "created":false
  }
  ```

  > 更新后, 版本`version`会自增. 状态由`created`变成`updated`
  
* 返回索引的类型下的所有文档

  ```bash
  $ curl 'localhost:9200/accounts/person/_search'
  
  {
    "took":2,
    "timed_out":false,
    "_shards":{"total":5,"successful":5,"failed":0},
    "hits":{
      "total":2,
      "max_score":1.0,
      "hits":[
        {
          "_index":"accounts",
          "_type":"person",
          "_id":"AV3qGfrC6jMbsbXb6k1p",
          "_score":1.0,
          "_source": {
            "user": "李四",
            "title": "工程师",
            "desc": "系统管理"
          }
        },
        {
          "_index":"accounts",
          "_type":"person",
          "_id":"1",
          "_score":1.0,
          "_source": {
            "user" : "张三",
            "title" : "工程师",
            "desc" : "数据库管理，软件开发"
          }
        }
      ]
    }
  }
  ```

  > `_score`为匹配度

# Query DSL

## 介绍

* ES搜索方式: URI Search 和 Query DSL. 主要以Query DSL为主, 类比SQL的Select

* Query DSL

  * 字段类查询
    * 单词匹配(Term Level Query) 不对查询的内容进行分词
    * 全文索引(Full Text Query) 对查询的内容进行分词, 如查询"我在马路边", 被分词为"我", "在", "马路"等, 然后再去匹配.
  * 复合查询

* 查询URL

  Query DSL查询的URL路径以`_search`结尾, 如

  查询索引的类型下的文档

  ```url
  localhost:9200/accounts/person/_search
  ```

  查询索引下的文档

  ```url
  localhost:9200/accounts/person/_search
  ```

  查询所有索引下的文档

  ```url
  localhost:9200/_search
  ```

## 字段类查询

[Elastic Search之Search API(Query DSL)、字段类查询、复合查询](https://blog.csdn.net/fanrenxiang/article/details/86477019)

### 全文匹配







### 单词匹配



## 复合查询



## 其他

* 简单查询形式

  ```bash
  $ curl 'localhost:9200/accounts/person/_search'  -d '
  {
    "query" : { "match" : { "desc" : "管理" }},
    "from": 1,
    "size": 1
  }'
  ```

  * `query` 为查询语句集合; `match`是一个具体的语句, 表示动作; `desc`表示动作的描述
  * `size` 查询多少条, 默认10条
  * `from` 从哪个位置开始查询, 默认0

* 逻辑运算

  多个关键次以空格隔开, 表示`or`关系

  ```bash
  $ curl 'localhost:9200/accounts/person/_search'  -d '
  {
    "query" : { "match" : { "desc" : "软件 系统" }}
  }'
  ```

  要执行`and`关系查询, 需使用布尔查询

  ```bash
  $ curl 'localhost:9200/accounts/person/_search'  -d '
  {
    "query": {
      "bool": {
        "must": [
          { "match": { "desc": "软件" } },
          { "match": { "desc": "系统" } }
        ]
      }
    }
  }'
  ```

# 其他

## Kibana

* 介绍

  Kibana 是一个开源的分析和可视化平台，旨在与 Elasticsearch 合作。Kibana 提供搜索、查看和与存储在 Elasticsearch 索引中的数据进行交互的功能。开发者或运维人员可以轻松地执行高级数据分析，并在各种图表、表格和地图中可视化数据。

* 运行

  ```shell
  docker run -d --name kibana -p 5601:5601 kibana:7.8.0
  ```

  打开http://localhost:5601 可访问到kibana
  
  > 注意, 还需要将elasticsearch和kibana加入到同一自定义bridge网络中, 否则容器间无法通信. (见[docker网络知识](https://sidian.live/article/?id=268#head-15-0-0-0-0-0))
  
* 基本使用

  * Index Patterns (*Management/Kibana/Index Patterns*)

    Index patterns定义匹配Index的模式, 聚集一组Index, 以便查询. 

    支持通配符`*`, 匹配0到多个字符

  * Discover

    查询文档, 这里用到了上述定义的Patterns. 

    还支持按字段查询.

* 参考

  * [Configure Kibana](https://www.elastic.co/guide/en/kibana/current/settings.html) kibana的配置
  * [Install Kibana with Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)

# 参考

* [Elastic Stack and Product Documentation](https://www.elastic.co/guide/index.html) 官方文档

* [全文搜索引擎 Elasticsearch 入门教程 阮一峰](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)
* [Elasticsearch简介与实战](https://www.jianshu.com/p/d48c32423789)

* [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl.html#query-dsl) 全文搜索语言
* [Elasticsearch(10) --- 内置分词器、中文分词器](https://www.cnblogs.com/qdhxhz/p/11585639.html)
* [ElasticSearch中文分词](https://www.jianshu.com/p/bb89ad7a7f7d)

* [elasticsearch教程--中文分词器作用和使用](https://blog.csdn.net/an88411980/article/details/83747230)
* [Elasticsearch全文检索入门这一篇就够了](https://zhuanlan.zhihu.com/p/94181307)