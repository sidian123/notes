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

  模式中用到的变量, 可在语句中产生.

  在`match`语句中, 未赋值的变量表示所有; 在`create`语句中, 未赋值的变量表示空

## 语句

* create 创建节点, 关系

  多个节点`,`分隔

  ```CQL
  CREATE (pattern)
  ```

  节点带有标签, 属性

  ```cql
  create (node:label {key:value,key2:value2}
  ```

  创建关系

  ```
  CREATE (node1)-[:RelationshipType]->(node2) 
  ```

* `return` 返回变量结果

  ```cql
  CREATE (Node:Label{properties. . . . }) RETURN Node 
  ```

  



## 函数









# 参考

* [Neo4j Documentation](https://neo4j.com/docs/)
* [Neo4j Tutorial](https://www.tutorialspoint.com/neo4j/index.htm)