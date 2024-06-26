# Hadoop概览


# HDFS

1. HDFS概述  
  
Hadoop 分布式系统框架中，首要的基础功能就是文件系统，在 Hadoop 中使用 FileSystem 这个抽象类来表示我们的文件系统，这个抽象类下面有很多子实现类，究竟使用哪一种，需要看我们具体的实现类，在我们实际工作中，用到的最多的就是HDFS(分布式文件系统)以及LocalFileSystem(本地文件系统)了。  
  
在现代的企业环境中，单机容量往往无法存储大量数据，需要跨机器存储。统一管理分布在集群上的文件系统称为分布式文件系统。  
  
HDFS（Hadoop  Distributed  File  System）是 Hadoop 项目的一个子项目。是 Hadoop 的核心组件之一，  Hadoop 非常适于存储大型数据 (比如 TB 和 PB)，其就是使用 HDFS 作为存储系统. HDFS 使用多台计算机存储文件，并且提供统一的访问接口，像是访问一个普通文件系统一样使用分布式文件系统。  
  

![6l0xa2.png](https://files.catbox.moe/6l0xa2.png)

  
  

2. HDFS架构  
  

![7srz72.png](https://files.catbox.moe/7srz72.png)

  
HDFS是一个主/从（Mater/Slave）体系结构，由三部分组成： NameNode 和 DataNode 以及 SecondaryNamenode：  
  
● NameNode 负责管理整个文件系统的元数据，以及每一个路径（文件）所对应的数据块信息。  
● DataNode 负责管理用户的文件数据块，每一个数据块都可以在多个 DataNode 上存储多个副本，默认为3个。  
● Secondary NameNode 用来监控 HDFS 状态的辅助后台程序，每隔一段时间获取 HDFS 元数据的快照。最主要作用是辅助 NameNode 管理元数据信息。  
  

![76lm4z.png](https://files.catbox.moe/76lm4z.png)

  
  
3. HDFS的特性  
  
首先，它是一个文件系统，用于存储文件，通过统一的命名空间目录树来定位文件；  
  
其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。  
  
1. master/slave 架构（主从架构）  
  
HDFS 采用 master/slave 架构。一般一个 HDFS 集群是有一个 Namenode 和一定数目的 Datanode 组成。Namenode 是 HDFS 集群主节点，Datanode 是 HDFS 集群从节点，两种角色各司其职，共同协调完成分布式的文件存储服务。  
  
2. 分块存储(Block)
  
HDFS 中的文件在物理上是分块存储（block）的，块的大小可以通过配置参数来规定，默认大小在 hadoop2.x 版本中是 128M。  
  
3. 名字空间（NameSpace）  
  
HDFS 支持传统的层次型文件组织结构。用户或者应用程序可以创建目录，然后将文件保存在这些目录里。文件系统名字空间的层次结构和大多数现有的文件系统类似：用户可以创建、删除、移动或重命名文件。 Namenode 负责维护文件系统的名字空间，任何对文件系统名字空间或属性的修改都将被 Namenode 记录下来。 HDFS 会给客户端提供一个统一的抽象目录树，客户端通过路径来访问文件，形如：hdfs://namenode:port/dir-a/dir-b/dir-c/file.data。  
  
4. NameNode 元数据管理  
  
我们把目录结构及文件分块位置信息叫做元数据。NameNode 负责维护整个 HDFS 文件系统的目录树结构，以及每一个文件所对应的 block 块信息（block 的 id，及所在的 DataNode 服务器）。  
  
5. DataNode 数据存储  
  
文件的各个 block 的具体存储管理由 DataNode 节点承担。每一个 block 都可以在多个 DataNode 上。DataNode 需要定时向 NameNode 汇报自己持有的 block 信息。 存储多个副本（副本数量也可以通过参数设置 dfs.replication，默认是 3）  
  
6. 副本机制  
  
为了容错，文件的所有 block 都会有副本。每个文件的 block 大小和副本系数都是可配置的。应用程序可以指定某个文件的副本数目。副本系数可以在文件创建的时候指定，也可以在之后改变。  
  
7. 一次写入，多次读出  
  
HDFS 是设计成适应一次写入，多次读出的场景，且不支持文件的修改。 正因为如此，HDFS 适合用来做大数据分析的底层存储服务，并不适合用来做网盘等应用，因为修改不方便，延迟大，网络开销大，成本太高。

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/bigdata/hadoop/hadoop%E6%A6%82%E8%A7%88/  

