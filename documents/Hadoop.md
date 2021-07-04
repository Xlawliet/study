# Hadoop

是Apache基金会所开发的分布式系统基础架构。

主要解决，海量数据的**存储**和海量数据的**分析计算**问题。

Hadoop生态圈：Zookeeper hive....

三大版本：

Apache 版本最原始的版本，对于入门学习最好

Cloudera 在大型互联网企业中用的较多

Hortonworks 文档较好



4高： 

高可靠性：底层维护多个数据副本，所以即时Hadoop某个计算元素或存储出现故障，也不会导致数据的丢失。

高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点。

高效性：在MapReduce的思想下，Hadoop是并行工作的，以加快任务处理的速度。

高容错性：能够自动将失败的任务重新分配。



1.x和2.x的区别：

<img src="D:\typora笔记\Hadoop.assets\image-20201023153746983.png" alt="image-20201023153746983" style="zoom:80%;" />





### HDFS

NameNode（nn）：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间，副本数，文件权限等），以及每个文件的块列表和快所在的DataNode等；



DataNode（dn）：在本地文件系统存储文件块数据，以及块数据的校验和；



Secondary NameNode（2nn）：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。



### YARN

ResourceManager（RM）主要作用如下

1. 处理客户端请求
2. 监控NodeManager
3. 启动或监控Application
4. 资源的分配与调度

NodeManager（NM）主要作用如下

1. 管理单个节点上的资源
2. 处理来自ResourceManager的命令
3. 处理来自ApplicationMaster的命令

ApplicationMaster（AM）作用如下：

1. 负责数据的切分
2. 为应用程序申请资源并分配给内部的任务
3. 任务的监控与容错

Container

是YARN中的资源抽象，封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等；



### MapReduce

将计算过程分为两个阶段：Map和Reduce

1. Map阶段并行处理输入数据
2. Reduce阶段对Map结果进行汇总

