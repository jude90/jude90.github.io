---
title: "推荐系统笔记 Item based 协同过滤 "
layout: post
date: 2015-11-14 15:01:00
---

### 

上周读了《推荐系统实践》，作者重点讲解了协调过滤，而在协同过滤中又着重讲了基于用户（User Based）和基于物品(Item Based) 两种算法。

两种算法非常相似，都是利用用户对物品偏好与相似度的加权。区别在于，基于用户的协同过滤利用的是其他用户对物品的偏好与用户间的相似度；而基于物品的协同过滤利用了一个用户对他评价过的物品与其他物品间的相似度来选择最佳推荐。

基于物品的协同过滤适用于用户比物品多，而且变化频繁的场景，这样不需要针对新用户重新计算相似度矩阵，只需要用户开始选择或者购买物品，就可以给用户推荐新物品。

基于用户的协同过滤适用于新闻推送这种物品更新频繁，用户数量相对稳定的情况。对于新闻这样强调时效性的物品来说，如果基于物品做协同过滤，物品数量线性增长时，相似度矩阵的大小却平方级地增长。

这两种协调过滤算法的优点，就是模型容易理解，解释力强, 非常符合直觉。而共同的缺点，恐怕就是计算开销了。所以Spark的mllib 模块只实现了一种推荐算法，ALS ,一种基于矩阵分解的机器学习算法。

但是Spark依然为那些想在Spark 上编写User/Item Based CF的人提供了一些有用的组件，比如利用RowMatrix，计算出相似度矩阵。

这周抽空写了一个ItemBased CF的demo 。算法本身是容易理解的，首先计算出相似度矩阵，然后对于用户的物品列表，加权求和，计算出用户可能会喜欢的N个商品。

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
        .filter{ case (item, pref)=> !uitems.contains(item) }
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

《推荐系统实践》里面用了Python 示意，用到了字典和嵌套的循环，这些在spark rdd 上都没法实现，考虑到分布式编程的特性，相似度矩阵和用户的物品列表都不可能全部放进内存作为一个字典随机地访问，都是基于RDD实现的。 所以代码也只好写成上面这样稍微有点难懂的rdd 操作。

现在的相似度矩阵完全依赖RowMatrix的API 来完成，无法相似度计算方法定制化，而且官方的api 也提供了一个 参数threshold，通过降低相似度矩阵的准确度来提升计算的效率 。有空深入理解分布式矩阵的实现以后再做改进。

[代码已经放在github](https://github.com/jude90/recomovies) 

参考：

[推荐系统学习：协同过滤实现](http://wuchong.me/blog/2014/04/19/recsys-cf-study/#)

[推荐系统中协同过滤算法实现分析](http://my.oschina.net/BreathL/blog/62519)

[用spark实现基于物品属性相似度的推荐算法](http://3iter.com/2015/10/12/%E7%94%A8spark%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E%E7%89%A9%E5%93%81%E5%B1%9E%E6%80%A7%E7%9B%B8%E4%BC%BC%E5%BA%A6%E7%9A%84%E6%8E%A8%E8%8D%90%E7%AE%97%E6%B3%95/)
