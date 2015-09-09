---
title: "hadoop 安装总结" 
layout: post
date: 2015-09-09 15:30:08
---

### 部署
每个大数据小白都是从 动手安装hadoop 开始他的大数据库之旅的。
- 第一步，在所有节点上创建相同的用户，比如叫hadoop。

- 生成 ssh 密钥，实现主节点到所有从节点的无密码登陆。hadoop 的很多操作依赖 ssh 登陆。因此这一步很关键。
- 写好hosts 文件，发送到所有节点。hosts 中主机名必须和该主机的hostsname 一致。否则可能出问题。在更大规模的部署中，DNS 服务器会代替 hosts 文件。
- 安装 JAVA。想让hadoop 正常启动 ，需要在 环境变量里 配置好JAVA_HOME 。平时我们装java ，会把 JAVA_HOME 写在 /etc/profile 里面，而hadoop 项目提供了脚本hadoop-env.sh 来收集原本散落各处，hadoop 运行所需的环境变量。
hadoop 的其他脚本在启动时都会执行hadoop-env.sh或者环境变量。 Hadoop 生态圈中的其他组件也遵循着相同的约定，比如hive 的conf/ 中有hive-env.sh ，spark的 conf/ 里也有spark-env.sh 

- 配置 slaves 文件，填写的内容是从节点的主机名，与hosts 文件内的名称一致。

接下来配置Hadoop的三个核心配置文件。也就是让hadoop能跑起来所需要的最基本的配置文件。
- core-site.xml
- hdfs-site.xml
- mapred-site.xml (在 hadoop 2.x 系列里面 已被 yarn-site.xml 取代。)

这三个文件的初始状态是一个空的 `configuration` 标签。
在 core-site.xml 里面，我配置了
`fs.default.name` 
`hadoop.tmp.dir`
`dfs.name.dir`
在 hdfs-site.xml 里面 配置了
`dfs.replication`
`dfs.data.dir`
在 mapred-site.xml 里面配置
`mapred.job.tracker`
`mapred.system.dir`
`mapred.local.dir`

可以看出，这些都是hadoop启动需要的关键参数，比如主节点服务的端口号。NameNode ,DataNode 数据存放的位置。然而如果不写这些参数，hadoop 也会使用默认的配置参数，比`hadoop.tmp.dir` 如果不设置，hadoop就会默认 /tmp 作为 hadoop的 临时目录。
还有 `dfs.name.dir`, `dfs.data.dir` 都有各自的默认配置。以上配置中似乎只有`fs.default.name`和 `mapred.job.tracker` 是必须配置的，否则不能正常启动。

由此可见，hadoop 遵循了一个重要且实用的原则：约定优于配置。整个hadoop 可能包含了数百个运行所需的配置项，但大部分配置只要使用默认值，就可以使hadoop 正常工作。

这也就是为什么 网上林林总总的hadoop 安装教程里面，三个核心配置文件的内容总是那么不一致。有的多了这个配置，有的缺了那个配置，给初学者带来困惑。
配置 hadoop ，让它正常工作，不跳出恼人的错误日志，也成了一门玄学。

当这些配置工作都完成之后，你就可以执行hadoop 的诸多脚本了。第一次启动 hdfs 前需要 执行 hadoo namenode -format 使文件系统初始化，然后执行 star-all.sh 启动所有的节点吧。


### 常见问题
**hosts**
配置文件是一门玄学，也是容易出问题的地方。作为分布式系统，配置文件的不一致更是要命。
最简单的不一致就是 hosts 文件的不一致。包括hostsname 与 hosts 文件内的名字不一致。
如果只有 主节点 有 hosts 文件，其他节点没有。集群也无法启动。

**pid 文件放哪儿**
集群在启动的时候，每个进程都会生成一个对应的pid 文件。这些pid 文件最初放在 /tmp 下面， 而启动时会有报错显示 hadoop 用户没有 /tmp 的写权限。导致启动失败。解决办法可以是 修改 hadoop 用户的权限，让他能够在 /tmp 目录写文件。也可以修改 hadoop-env.sh 中的参数HADOOP_PID_DIR ，指向hadoop用户可写的路径。比如/home/hadoop/tmp

**HDFS error: could only be replicated to 0 nodes, instead of 1**
当HDFS正常启动，我迫不及待地 上传了一个文件到hdfs 来验证它是否工作正常。
简直是荒唐，所有节点都启动正常。NameNode ，DataNode 进程都活着，为什么会出现这样的错误？
爆栈的解释是，可能存在以下情况：

 - 数据节点磁盘用完了
 - 数据节点正在进行块扫描
 - 配置文件错误，如 dfs.block.size 被设置成负数。
 - 等等等等。。 

然而解决方法却异常的简洁： `service iptables stop`.
当我关掉了 所有数据节点的 iptables，文件就正常地上传了，没有报错。（数据节点都是 redhat6.5）以后安装好hadoop 以后还得记得关掉防火墙。