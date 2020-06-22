* 介绍

  * 以节点, 关系和属性的形式存储数据.

  * 属性图模型(Property graph model )规则
    * 表示节点，关系和属性中的数据
    * 节点和关系都包含属性
    * 关系连接节点
    * 属性是键值对
    * 节点用圆圈表示，关系用方向键表示。 
    * 关系具有**方向**：单向和双向。 
    * 每个关系包含“开始节点”或“从节点”和“到节点”或“结束节点” 
  * Cypher查询语言

* 安装

  ```shell
  docker run \
      --publish=7474:7474 --publish=7687:7687 \
      --volume=$HOME/neo4j/data:/data \
      neo4j
  ```

  启动后, Neo4j提供了一个访问界面 http://localhost:7474/

  