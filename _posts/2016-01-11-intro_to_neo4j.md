---
title: "Neo4j 笔记 1 基础概念" 
layout: post
date: 2016-01-11 09:14:00
---


### Node & Relation 

图数据库的两个基本概念：

- 节点（Node)
- 关系（Relations, 也就是图的边）

在 neo4j 里面 节点用 `()` 表示，例如：

- `(a)` 一个叫 a 的节点
- `( )` 一个匿名节点
	
neo4j 的Relation 用 ` -[ooxx]->` 来表示，例如：

- `-[r]->` 一个叫做 `r`的关系（边）。
- `(a)-[r]->(m)` 演员和电影 间 存在一个 叫做 `r` 的关系。
- `-[:ACTED_IN]->` 一个`ACTED_IN` 类型的关系。
- `(a)-[:ACTED_IN]->(m)` 对 某些电影有 `ACTED_IN` 关系的演员 
- `(d)-[:DIRECTED]->(m)` 对某些电影有`DIRECTED`关系的导演。

### Properties

节点和关系 都可以拥有 properties 。properties 是键值对，用来给节点和关系 添加一些属性信息。比如 `演员`j节点可能有一个`name`属性和一个`born`属性表示出生年份；`电影`节点 可能会包含 一个`title` 属性。

节点的属性表达方式如下：
 	
- `(m {title:"The Matrix"})` : 一个带有 `title`属性的 电影
-  `(a {name:"Keanu Reeves",born:1964})`: 一个带有名字和出生年份的演员

关系也可以带有属性
 `(a)-[:ACTED_IN {roles:["Neo"]}]->(m)`  带有roles属性(一个字符串列表)`ACTED_IN`关系。

 **备注**: 属性的类型可以是布尔型，数字或字符串，也可以是包含上面三种类型的列表。


### Labels
Label 用来给 节点 标注类型，用以区分不同类型的节点。一个节点可以有一个或者多个Label，例如：

- `(a:Person)` 一个Person 类型的节点
- `(a:Person {name:"Keanu Reeves"})`一个带有属性的Person类型节点 
- `(a:Person)-[:ACTED_IN]->(m:Movie)` 一个Person类型的节点跟 一些Movie 类型的节点有`ACTED_IN`关系。

