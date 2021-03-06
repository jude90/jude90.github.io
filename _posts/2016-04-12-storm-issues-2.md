---
title: "问题清单2016.4.11" 
layout: post
date: 2016-04-12 14:51:47
---

上周遇到了几个大坑，让我对storm的理解受到了严峻的挑战。
问题的起因很简单。我在kafka 队列里面输入了一点非法字符，也就是非json 模式的字符串。当bolt 尝试将脏字符串处理成 JSONObject 时会抛出一个异常。之前一直没有catch 这个异常，觉得数据的提供者应该保证 kafka 里面都是干净的数据。另外，当时我认为一条引发异常的tuple 会被 storm 扔掉。接下来的干净的tuple 还是可以继续正常处理的。
但是。自从那几条脏数据被丢进kafka，一切就开始不对劲了。StormUI 上的统计指标开始乱套。Errors 列表上 始终在报错，而且都是json 解析异常。尽管脏数据后面还有几百条干净数据。但是数据流似乎被阻塞在了抛出异常的Bolt上，下游Bolt的tuple处理数量都是0，而json解析抛出的异常数量比如 脏数据本身还要多。Storm并没有丢弃引发异常的tuple，反而抓着不放。log 里面也看不到写入 els 的记录。

解决问题本身并不复杂。把异常接住就行。重要的是它改变了我之前的一些看法。

kafka 里的数据是不可靠的，可能存在脏数据。storm不对自动丢弃引发异常的脏数据。Bolt execute 函数抛出的异常会使 下游的Bolt 无法得到数据。整个Topology陷入瘫痪。

还有一些经验是关于 Storm 编程本身的。Bolt 什么时候接收了数据，什么时候调用 execute 函数去处理tuple，collector emit 之后发生了哪些事情。这些东西我都没法去了解。因为缺乏全局的把握，所以一旦遇到问题就很容易陷入焦虑。接住异常就解决问题只是侥幸。打印日志是一个好办法，写单元测试也是。Storm starter 里面提供了一些基于mockito 的单元测试，供我们依葫芦画瓢。



