---
title: " Neo4j 笔记 2 Cypher" 
layout: post
date: 2016-01-14 01:56:53
---



Cypher 是neo4j 的查询语言，正如SQL 之于 关系型数据库。可以实现select，insert， update， delete。

图的声明语句也是 Cypher 的一部分，如
`(a) -[:LIKES]-> (b)`

### 概述
Cypher 的语法看起来有点像SQL ，支持表达式，where 从句。一个 Cypher 语句 通常是从 MATCH 关键字开始的。结构如下：

```
MATCH (node) RETURN node.property
MATCH (node1)-->(node2) 
RETURN node2.propertyA, node2.propertyB
```

Cypher 运用模式匹配来搜索节点和关系。如果我想找到任何 由 ACTED_IN 关系 连接起来的两个节点，需要这样写：
```
MATCH (actor)-[:ACTED_IN]->(movie)
RETURN actor, movie 
```

这里的 actor 和 movie 作为变量名，存储了连接在ACTED_IN 关系两端的 节点(Node)。也可以在RETURN 从句里面 访问 变量的数据。根据 一个指定关系(Relation ) 来匹配 节点的 通用写法如下：

```
MATCH (node1)-[:REL_TYPE]->(node2)
...

```

neo4j 的Relation 也是图模型的一部分，它也可以携带属性。如果想要得到关系的属性，要用一个变量名来表示关系。通过`[rel:TYPE]`的匹配方式，每次匹配到的关系对象会绑定到该变量上，以便我们访问它的属性。


```
MATCH (node1)-[rel:TYPE]->(node2) 
RETURN rel.property
```

上面我们看到了 匹配特定类型 `关系(Relation)` 的语法。其实neo4j 中节点和关系都拥有类型。所以节点和关系都可以用  类型标签来进行 模式匹配 ， 通常的写法如下。


```
MATCH (node:Label) RETURN node

MATCH (node1:Label1)-[:REL_TYPE]->(node2:Label2) 
RETURN node1, node2
```

