---
title: "键值型 RDD" 
layout: post
date: 2015-10-20 15:12:32
---
spark 会自动地将包含二元组的RDD看做PairRDD，并且针对key-value 型数据的特性和应用场景进行了性能优化。

`combineByKey`  是PairRDD里所有聚合函数的一般形式，其他的聚合函数都由combinByKey 来实现。
要理解combinByKey的过程，就得理解 它是如何处理每个元素的。考虑一下当遍历某个分区的每个元素时，每个元素的key 会面临两种情况：之前出现过，或者第一次出现。

每当迭代到新出现的key ，combineByKey() 会调用用户提供的回调函数create
Combiner()，为这个新Key 创建一个新的累加器。注意，这是在一个Partition 内部迭代的，而不是在横跨多个Partition的RDD 上迭代。所以每个Partition 都为每个key 创建独立的累加器(Combiner)

每当迭代到之前处理过的key, combineByKey() 会调用另一个回调 mergeValue()，由它来实现 元素的value合并到累加器的过程。

用一个简单的 wordcount 描述上述过程，大体上是这样

```python
accumulators = dict()
for word in words:
    if accumulators.has_key(word):
        accumulators[word] += 1
    else:
        accumulators[word] = 1
```

在上例中 ，累加器就是一个数字，所以合并value到 累加器的计算，也就是简单的加1。 如果累加器是一个容器，比如列表，那么合并函数可以是向列表里存放value ,或者对列表做其他操作。

在Spark中，每个Partition的计算过程是并行的，相同的key 会在多个累加器，当我们最终合并每个Partition中key 相同的累加器时，就要用到第三个回调函数mergeCombiners()

### Partitions , Partitioner
数据分区是分布式系统的一个核心概念，起初是因为数据体积庞大到单机无法处理，而当数据分布在不同节点之后，跨节点的计算和数据迁移带来了很大开销。
如果你有两个key value 型的RDD，要用join 得到一个新的RDD，那么每次计算的时候，都需要将两个数据集中间含有相同key 的元素发送到同一个节点上。这时候分区设计的好坏（partitioning）对于执行效率会产生显著的影响。

虚基类Partitioner 声明了变量numPartitions 和函数   getPartition，函数接收任意类型的key, 返回一个Int， 代表分区的编号。

几乎所有PairRDD的方法都有一个版本，允许API使用者传递参数numPartitions，显式地指定PairRDD的Partition数量。用numPartitions，可以创建一个Partitioner 的子类， 比如最常见的HashPartitioner。
HashPartitioner 顾名思义，就是把key 的哈希值模分区数，得到这个 key的分区号。以ByKey 结尾的transformation， 生成ShuffledRDD , 代表spark 执行的stage 边界，当执行到此处时，数据被分别计算分区编号，然后发送到对应的节点，数据在集群内完成一次迁移。这是一个开销很大的操作，而之后的计算里，相同key的values 存在对应的partition内，计算本地化，开销减小。

**partitioner**还可以从父RDD处传递给子RDD， 在reduceByKey() 之类的 transformation 内部，程序先检查父RDD的partitioner是否存在，是否与当前的partitioner参数相同。如果两个条件都满足，那说明RDD已经执行了分区，直接执行 mapPartitions ，否则就new ShuffledRDD, 标记stage 。显然如果之前有个一次分区，后面使用同类型的 ByKey 计算都会很快。

参考

learning spark 第四章