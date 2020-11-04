# 介绍

## 基本概念

* Nodes - graph data records
* Relationships - connect nodes
* Properties - named data values

## 属性图模型

Neo4j遵循*属性图模型*(Property graph model )规则

* 将数据表示在节点，关系和属性中

* 节点和关系都包含属性

* 关系连接节点

* 属性是键值对.

  > 在Neo4j中, 值可以是string,number,boolean或array类型的.

* 节点用圆圈表示，关系用方向键表示。 

* 关系具有**方向**：单向和双向。 

  > 在Neo4j中, 创建关系必须单向的, 但查找时可以双向查找.

* 每个关系包含“String Node”或“From Node”和“To Node”或“End Node” 

## 图数据库组成

* 节点
* 关系
* 属性

* 标签
  * A node can have zero or more labels
  * Labels do not have any properties
  * 标签用于归纳节点, 形成组, 可类比于关系数据库中的表
* 关系
  * Relationships always have direction
  * Relationships always have a type
* Cypher查询语言 (CQL)

## 安装

```shell
docker run \
    --publish=7474:7474 --publish=7687:7687 \
    --volume=$HOME/neo4j/data:/data \
    neo4j
```

启动后, Neo4j提供了一个访问界面 http://localhost:7474/  

`7687`是数据的访问端口

默认账号密码: `neo4j`/`neo4j`

# CQL

## 匹配模式

匹配模式用于描述节点和关系, 用于具体的操作语句中.

* 节点模式

  ```CQL
  ()   // 表示匿名的节点
  (matrix) // matrix为节点模式变量 
  (:Movie) // :Movie表示节点标签为Movie
  (matrix:Movie)
  (matrix:Movie {title: "The Matrix"}) // {...}声明节点属性
  (matrix:Movie {title: "The Matrix", released: 1997})
  ```

* 关系模式

  ```CQL
  --> // 匿名的关系, 方向右
  -- // 匿名的关系, 无向
  -[role]-> // role为关系模式变量
  -[:ACTED_IN]-> // :ACTED_IN表示关系类型
  -[role:ACTED_IN]->
  -[role:ACTED_IN {roles: ["Neo"]}]-> // {...}声明关系属性
  ```

* 变量

  节点模式声明中可声明变量, 如上所示; 也可将整个模式赋值给变量, 如下

  ```
  acted_in = (:Person)-[:ACTED_IN]->(:Movie)
  ```
  
  > 在`match`语句中, 未赋值的变量表示所有; 在`create`语句中, 未赋值的变量表示空; 执行后, 变量会被赋值.

## 语句

* 创建节点

  多个节点`,`分隔

  ```CQL
  CREATE (pattern)
  ```

  节点带有标签, 属性

  ```cql
  create (node:label {key:value,key2:value2}
  ```

* 创建关系

  ```cql
  CREATE (node1)-[:RelationshipType]->(node2) 
  ```

  > 若节点不存在, 会创建

  为已知节点创建关系, 需要通过`match`获取节点

  ```cql
  MATCH (a:LabeofNode1), (b:LabeofNode2) 
     WHERE a.name = "nameofnode1" AND b.name = " nameofnode2" 
  CREATE (a)-[: Relation]->(b) 
  RETURN a,b 
  ```

* `return` 返回变量结果

  ```cql
  CREATE (Node:Label{properties. . . . }) RETURN Node 
  ```

* `match`

  根据节点或关系信息过滤出**图**来

  ```cql
  MATCH (n:player)  // 查询单个节点
  MATCH (node:label)<-[: Relationship]-(n)  // 提供后继节点和关系, 查询图. 其中查询出的前继节点以n表示
  MATCH (n) RETURN n // 匹配所有节点, 返回图
  MATCH p=()-[r:BATSMAN_OF]->() RETURN p LIMIT 25
  ```

* `merge`

  给定模式, 现在图中搜索, 若找到, 则返回, 否则创建.

* `delete`

  删除全部节点和关系

  ```cql
  MATCH (n) DETACH DELETE n
  ```

* 其他

  * delete 删除
  * where 缩小模式匹配范围
  * limit 限制显示个数
  * ...

## 函数

* `count(n)` 计算变量个数

## 索引 & 约束

# 参考

* [Neo4j Documentation](https://neo4j.com/docs/)
* [Neo4j Tutorial](https://www.tutorialspoint.com/neo4j/index.htm)

# -----学习二------

# Introduction

* Cypher由一个链式从句组成, 一个从句的临时结果作为下一个从句的输入

* 常用查询命令
  - `MATCH`: The graph pattern to match. This is the most common way to get data from the graph.
  - `WHERE`: Not a clause in its own right, but rather part of `MATCH`, `OPTIONAL MATCH` and `WITH`. Adds constraints to a pattern, or filters the intermediate result passing through `WITH`.
  - `RETURN`: What to return.
  
* 常用更新命令
  * `CREATE` (and `DELETE`): Create (and delete) nodes and relationships.
  * `SET` (and `REMOVE`): Set values to properties and add labels on nodes using `SET` and use `REMOVE` to remove them.
  * `MERGE`: Match existing or create new nodes and patterns. This is especially useful together with unique constraints.

* 基本概念

  * **DBMS** Neo4j数据库的管理系统, 可包含多个数据库
  * **Database** 数据库, 客户端会话连接的对象. 可包含多个图谱, 但初始情况下仅含一个图谱.
  * **Graph** 图谱, 图谱查询更新语句操作的对象. 一个数据库中存在多个图谱时, 查询语句可指定图谱, 或使用默认图谱. 

  初始情况下, 一个DBMS中含有两个数据库:

  * `system` 系统数据库, 含有配置信息. 仅管理员命令可操作该数据库.
  * `neo4j` 默认数据库, 含有一个默认图谱.

  > 开源版本Neo4j只能含有以上两个数据库, 不支持更多的数据库了.

* 事物

  * 所有语句都运行在事物中, 默认一条语句一个事物. 可手动开启事物, 让多个语句处于同一个事物中, 并提交.
  * 事物是数据库级别的, 即开启事物锁定整个数据库. 同时还有一些限制, 略.

  


# 疑问点

* with 的作用