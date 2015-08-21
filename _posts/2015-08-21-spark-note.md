---
title: "spark 笔记1" 
layout: post
date: 2015-08-21 17:41:14
---

### 概览
一个spark 应用包括了：

- 一个`driver` ， 运行在主节点，调度工作流程
- 若干 `executor` ， 分布在集群的节点上。执行任务，接收调度。
- `task`: 任务的基础单元，一个 `executor` 内部会运行多个`task`

以上三个概念只是spark 中间最基础，比较容易理解的部分，在Hadoop 里面也有与之类似的概念。
而spark中还有很多术语需要你去理解，他们看起来并非字面含义那么简单。

**Job** 
 Job 是一个顶层的概念，可以把spark-submit 提交的一个jar 包内执行的所有计算过程看成一个job. 当jar 包内部 spark api 中的 actions 类型的函数被调用时。spark 就会启动一个job , 通过分析对 RDD 的操作形成的图结构， 生成一个执行计划。这个过程很像数据库的查询规划。

**stage**
执行计划 会将 Job 所包含的一连串 transformation 划分成若干个stage.  在每个 stage 阶段内，spark 在所有数据的分区上执行相同的计算步骤。 而stage 的分界与否，取决于 这些 transformation 是否要将数据分区 打乱，重新发送和接收数据。

所谓重新发送和接收数据，也就是 hadoop 和spark 中所谓的 `shuffle` 动作。这种状态下，每个节点上的计算都会依赖其他数据节点的数据，同时也被别的节点依赖。于是这些计算过程里产生的中间数据，需要在整个集群上迁移，带来巨大的网络和磁盘读写，对于性能是很大的损失。

那么在spark里，什么样的transformation 会 带来 `shuffle`操作？

我们知道 `RDD` 是一个数据集，里面的内容也就是一条条数据记录。当`RDD`执行 `map` 或者 `filter` 这样的 transformation 时， 其生成 的`RDD`的数据分区，仅仅依赖于原`RDD`中相应的数据分区。如此变换形成的关系称为 **窄依赖**。
打个比方，子`RDD`分区 `C1` 的数据 仅仅依赖其父`RDD`分区 `B1` 的数据，对于`map` 变换是一一对应的关系；对于`filter`变换是过滤，都不会从其他分区引入数据。
而像`groupByKey` `reduceByKey`  这样的变换，需要使得RDD 分区按照`key`的值重新分布。这时候 spark 就会执行一个 `shuffle`动作。并且生成一个新的`stage` , 新的 `RDD`分区。这种变换而来的关系，称为**宽依赖**。
例如下面这段操作

```scala
sc.textFile("someFile.txt").
  map(mapFunc).
  flatMap(flatMapFunc).
  filter(filterFunc).
  count()
```

它只执行了一个 `action` 也就是 `count()`, 这些变换属于一个 `stage`, 因为这三个操作都不需要依赖其他分区 的数据。

相反，下面这段代码就被分成了三个 `stage`

```scala
val tokenized = sc.textFile(args(0)).flatMap(_.split(' '))
val wordCounts = tokenized.map((_, 1)).reduceByKey(_ + _)
val filtered = wordCounts.filter(_._2 >= 1000)
val charCounts = filtered.flatMap(_._1.toCharArray).map((_, 1)).
  reduceByKey(_ + _)
charCounts.collect()
```

两个 `reduceByKey` 把整个计算过程 分成了三个部分，它们根据RDD 的键 将数据重排， 生成新的RDD分区。

在 `stage`的边界，也就是`groupByKey`执行之处，数据会被上一个`stage`的 `task` 写入磁盘，并且通过网络被下一个`stage`的`task`读取。因为会带来严重的磁盘和网络IO 开销，`stage`的边界是昂贵的，需要尽量避免。`stage`之间 RDD的分区数量通常不一样，所以可能触发`stage`边界的transformation 函数都会提供一个`numPartitions`参数来让你觉定在下一个`stage` 使用多少个数据分区。

参考
[how to tune your spark jobs](http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-1/)
