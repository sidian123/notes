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



# 参考

* [Neo4j Documentation](https://neo4j.com/docs/)
* [Neo4j Tutorial](https://www.tutorialspoint.com/neo4j/index.htm)