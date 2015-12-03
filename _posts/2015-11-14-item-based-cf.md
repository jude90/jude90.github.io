---
title: "推荐系统笔记 Item based 协同过滤 "
layout: post
date: 2015-11-14 15:01:00
---


上周读了《推荐系统实践》，作者重点讲解了协调过滤，而在协同过滤中又着重讲了基于用户（User Based）和基于物品(Item Based) 两种算法。

### Item/User Based
两种算法非常相似，都是利用用户对物品偏好与相似度的加权。区别在于，基于用户的协同过滤利用的是其他用户对物品的偏好与用户间的相似度；而基于物品的协同过滤利用了一个用户对他评价过的物品与其他物品间的相似度来选择最佳推荐。

基于物品的协同过滤适用于用户比物品多，而且变化频繁的场景，这样不需要针对新用户重新计算相似度矩阵，只需要用户开始选择或者购买物品，就可以给用户推荐新物品。

基于用户的协同过滤适用于新闻推送这种物品更新频繁，用户数量相对稳定的情况。对于新闻这样强调时效性的物品来说，如果基于物品做协同过滤，物品数量线性增长时，相似度矩阵的大小却平方级地增长。

这两种协调过滤算法的优点，就是模型容易理解，解释力强, 非常符合直觉。而共同的缺点，恐怕就是计算开销了。所以Spark的mllib 模块只实现了一种推荐算法，ALS ,一种基于矩阵分解的机器学习算法。网上搜到的博客，也大多基于官网的例子。于是又有机会练习一下灵活（yin dang）的Scala啦 。

这周抽空写了一个ItemBased CF的demo 。算法本身是容易理解的，计算出相似度矩阵，然后对于用户的物品列表，加权求和，计算出用户可能会喜欢的N个商品。

推荐物品的Spark实现如下

```scala
def RecommendTop(userprefs:SparseVector,
                   similar:RDD[(Int, SparseVector)],
                   topk:Int =10,nums:Int=10)(implicit sc:SparkContext): Array[(Int,Double)] ={

    // key is items , values is preference of items
    val ipi =userprefs.indices zip userprefs.values
    // items that user has selected
    val uitems = userprefs.indices
    val ij = sc.parallelize(ipi)
      .join(similar)
      .flatMap{ case (i, (pi, vector:SparseVector)) =>
        (vector.indices zip vector.values)
        // filter item which has been in user's item list
        .filter{ case (item, _)=> !uitems.contains(item) }
        // sort by Sim(i,j) and then take top k
        .sortBy{ case (j, sij) => sij}.reverse.take(topk)
          // item j with Sim(i,j) * Pi
          .map{ case (j, sij) => (j, sij * pi) }
      }
    // reduce by item j , select top nums Preference
    val result = ij.reduceByKey(_+_).sortBy(_._2,ascending = false).take(nums)

    result
  }
```

《推荐系统实践》里面用Python 示意，使用了字典和嵌套的循环，这些单机写法都不适用于Spark RDD。由于分布式编程的特性，相似度矩阵和用户的物品列表不可能全部放进内存作为一个字典随机地访问，而是基于RDD实现，惰性求值的。物品列表和相似度矩阵想要加权求和，要通过join将两个RDD 归并到一起。归并之后生生的键值对代表了（备选物品 j， 偏好i × 相似度ij ），使用reduceByKey(\_+\_) 将键值对累加，结果就是（备选物品 j , 用户对物品 j 的偏好估计）。再做排序，抽出排名前N个物品，就得到了推荐物品列表。

现在的相似度矩阵完全依赖RowMatrix的API 来完成，无法相似度计算方法定制化，而且官方的api 也提供了一个 参数threshold，通过降低相似度矩阵的准确度来提升计算的效率 。有空深入理解分布式矩阵的实现以后再做改进。


### 推荐系统评价指标
实现完推荐系统后，就该跑点数据回测一下，验证系统的工作能力了。
对于一个推荐系统，可以从以下几个指标来评价：

- 推荐准确率(precision) ,推荐物品的命中率，它等于 推荐物品中命中用户物品列表的数量/ 推荐物品数量
- 召回率(recall),感觉这个翻译实在是蛋疼。它的真是含义是，推荐物品中命中用户物品列表的数量/ 用户物品列表的数量。
- 覆盖率: 表示推荐的物品占了物品全集空间的多大比例。
- 新颖度: 新颖度是为了推荐长尾区间的物品。用推荐列表中物品的平均流行度度量推荐结果的新颖度。如果推荐出的物品都很热门，说明推荐的新颖度较低，否则说明推荐结果比较新颖

![准确率与召回率](http://img.my.csdn.net/uploads/201106/14/0_1308034676G5GQ.gif)

**Precision/Recall**

为了计算提高效率 准确率和 召回率用实现在了同一个函数里面

```scala

  def PrecisionAndRecall(test_set:Array[IndexedRow],
                          K:Int=10,N:Int=10):(Double,Double)={
                          
    var recall:Double = 0
    var precision:Double = 0
    var hit:Double = 0
    test_set.map{ case IndexedRow(index,vector) =>(index, vector.asInstanceOf[SparseVector])}
      .foreach{ case (uid,  vect)=>
        // shuffled  item list before split
        val shuffled = Random.shuffle(vect.indices zip vect.values).asInstanceOf[Array[(Int,Double)]]
        // split a item vector into two exclusive part
        val (predict_set:Array[(Int,Double)], vali_set:Array[(Int,Double)]) = shuffled.splitAt(vect.indices.length.toInt/2)

//        reform the predict_set into a SparseVector
        val predicts =Vectors.sparse(predict_set.length, predict_set.toSeq).asInstanceOf[SparseVector]

        val recommends = RecommendTop(predicts,K,N)
        // hits mean intersection of test list and recommend list
      val hits = vali_set.indices.toSet intersect recommends.map(_._1).toSet
      hit += hits.size
      recall += vali_set.size
      precision += N

    }
    ( hit/recall, hit/precision )
  }
```

这个函数做的事情很简单，但是花了我好一会儿才写对。最初我对用户的物品列表求推荐列表，得到推荐列表后在与物品列表求交集。把交集的数量作为推荐命中数量。然后问题来了，推荐算法会过滤掉用户列表里面已存在的物品。如果验证集与预测集相同，预测的结果将无法命中验证集（命中的都被过滤啦），也就无法检验推荐算法的推荐效果。所以需要将用户列表随机拆分成两个不重叠的子集。分别作为推荐算法的输入，以及推荐结果的验证集。


[代码已经放在github](https://github.com/jude90/recomovies) 

参考：

[推荐系统学习：协同过滤实现](http://wuchong.me/blog/2014/04/19/recsys-cf-study/#)

[推荐系统中协同过滤算法实现分析](http://my.oschina.net/BreathL/blog/62519)

[推荐系统评测指标](http://bookshadow.com/weblog/2014/06/10/precision-recall-f-measure/)

[准确率 召回率 ](http://blog.csdn.net/wangzhiqing3/article/details/9058523)