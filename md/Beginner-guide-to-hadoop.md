## hadoop 初学者指南##
    这篇文章写给对零基础的初学者.不讲具体的开发,只是介绍整个体系的基础知识.
    首先,Hadoop是一个Apache软件基础项目,它提供了:
1. 一个分布式文件系统HDFS.
2. 一套软件框架和API用来创建和运行mapreduce任务.

###HDFS###
HDFS设计成一个类似标准Unix文件系统的分布式文件系统,数据被分散地存储在多台机器上.
设计HDFS不是为了替代标准的文件系统,而是为了提供一个用起来和标准文件系统差不多的分布式文件系统.它内置了针对机器宕机的处理机制,对吞吐量而非延迟进行了优化.

在一个HDFS集群中有两个半类型的机器:
- 数据节点 HDFS实际存放数据的地方,一般有好几个.
- 元数据节点 也就是'主'节点,它管理了整个集群的元数据,比如哪些数据块组成了某个文件,而这些数据块又存储在哪些数据节点上.
- 辅助元数据节点 它并不是备用主机,而是一台提供备份服务的机器,备份包括操作日志,文件系统的镜像,并且周期性地将日志合并到镜像以保证文件处于一个合适的大小.以后它会被专门的备用节点和存盘检查节点替代,但是功能上不会有什么区别.

你可以通过JAVA API或者 hadoop 命令行工具来访问数据.许多文件系统操作都是从Unix移植过来的.来看几个简单的例子:
列出根目录下的文件
`hadoop fs -ls /`
列出home 路径下的文件
`hadoop fs -ls ./`
输出一个文件(如果需要会解压缩)
`hadoop fs -text ./file.txt.gz`
上传或者下载一个文件
```
hadoop fs -put ./localfile.txt /home/matthew/remotefile.txt

hadoop fs -get ./home/matthew/remotefile.txt ./local/file/path/file.txt
```
请注意hadoop相比传统文件系统在某些方面做了优化,它适用于对吞吐量要求很高的非实时应用,而非对低延迟有很高要求的实时应用.比如文件一旦写入就不能够修改,而且文件读写的延迟相比标准文件系统也很糟糕.另一方面,吞吐量的规模可以随着集群节点数的增加而
线性地扩展,所以可以处理单机无法完成的工作量.

HDFS也有一些特性让它更适用于分布式环境.

- **容错机制** 数据被复制到多个节点以防止机器故障,业内公认的标准是复制3份(所有数据都会分配到三台机器上).

- **扩展性** 数据迁移直接在数据节点上进行,所以读写的兼容性很容易扩展到很多数据节点上. 
- **存储空间** 想要更多存储空间只需要买更多的数据节点就行.
- **行业标准** 很多分布式应用都基于HDF(HBase , Map-Reduce) 
- **和Mapreduce 配合默契**

###MapReduce###
Hadoop基础的第二部分就是 MapReduce 模块, 由两个组件构成.
- 