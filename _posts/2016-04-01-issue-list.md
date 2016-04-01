---
title: "问题清单1" 
layout: post
date: 2016-04-01 02:16:40
---

###问题清单 20160310

- [Maven mirror 与repository的区别](http://my.oschina.net/sunchp/blog/100634) 之前maven的repository和mirror 一直搞不清。最近要从公司的私服上下载一个jar包，依葫芦画瓢在 .m2/settings.xml 里面加了一个mirror 标签。放在原先 osc 的mirror 下面。打包的时候有报错，找不到私服的包，于是调换了两个mirror 的位置，结果私服的jar包可以找到，但是maven 公共仓库的包反而找不到了。百度一下，原来两个mirror的 mirrorOf 都写成了 星号。把他们改成 central，遂正常，私服和公共仓库都能找到。 因为mirrorOf 配置了星号会导致它成为唯一指定镜像库，显然不是我想要的。公司的私有仓库一般放到都是自己写的jar包，应该写到repository 标签里面。
- [kafka Failed to send messages after 3 tries](http://blog.csdn.net/zxcvg/article/details/18218483)  kafka client 连接失败，发现我属于第三种情况，server.properties 里面没有配置advertised.host.name , 修改之后就好了。
- [Kafka 自动关闭](https://segmentfault.com/q/1010000004292925)  吃个饭回来发现Storm topology挂了，一查发现是kafka 挂了把storm topology 带挂了。但是kafka日志上看不出什么报错。后来搜到这个问答。远程登录的控制台超时，控制台启动的kafka 也就挂了。以后要用nohup 启动。

- [java.lang.RuntimeException: java.lang.ClassCastException: java.lang.Long cannot be cast to java.lang.String](http://stackoverflow.com/questions/31139119/java-lang-runtimeexception-java-lang-classcastexception-java-lang-long-cannot) 改了一个topolog ，local模式下没有报错。但是submit 后storm ui 上显示错误：java.lang.RuntimeException: java.lang.ClassCastException: java.lang.Long cannot be cast to java.lang.String java.lang.backtype.storm.utils.DisruptorQueue.consumeBatchToCursor(Disruptor ... 搜了一下这个错。把topology conf 里面的Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS 配置注释掉。再提交到集群，topology 正常运行，不再有报错。

### 20160322 update
- 找不到主类。执行 storm jar , 提示找不到主类，没有其他的异常。唯一的改动是新增了slf4j 的依赖。去掉依赖后问题不再重现。原因是 storm lib 里面已经包含了 log4j 和 slf4j 的jar包。所以pom 里面 kafka 和 kafkaSpout 都会 将log 先关的依赖 exclude 掉。

- NullPointerException: MetricBolt 里面定义了一个logger。为了在构造函数里面根据被装饰的Bolt类确定日志输出名字，没有把logger 声明为private static final 。只声明了`private static logger;` 本地模式下测试，没有异常。但是提交到集群后，运行到打印处就会报错。原因不明，暂且记下。

### 20160330 update
- 部署storm 到3个节点时，遇到一个搞笑的问题。在新增的节点上启动supervisor后，去storm UI上查看supervisor，只能显示一个节点。这还不是重点，重点是每次刷新UI，supervisor 都会显示不同的节点名，但只有一个固定的supervisor ID。多个节点占用了相同的id ? 查看 zookeeper 上 /storm/supervisors 的内容，确实只有一个id 。谷歌了一下，可能跟三个节点上 storm-local 目录内容相同有关系,里面可能包含了相同的id号。删掉 其中两个节点的 storm-local ，重启 supervisor, storm UI上 显示出三个节点，各自拥有不同的ID. 
问题的原因大概是我把一个已经运行过的storm 目录直接发到了其他两个节点。storm-local  保存了一些运行状态，产生了id 冲突。

