# HDFS文件读写过程


&lt;!--more--&gt;
&gt; HDFS
## 文件的写入过程

HDFS把文件的数据划分成若干个块(Block)， 每个Block存放在一组 DataNode上，Namenode负责维护文件--&gt;Block(命令空间映射)和Block--&gt;Datanode(数据块映射)

![8pqx8k.png](https://files.catbox.moe/8pqx8k.png)

写入流程

1. Client发起文件上传请求，通过RPC与NameNode建立通讯，NameNode检查目标文件是否已存在，父目录是否存在，返回是否可以上传
2. Client请求确定第一个block 应该传输到哪些Datanode服务器上
3. NameNode根据配置文件指定的备份数量以及机架感知原理进行文件分配，返回可用的 DataNode地址，如：A，B，C
&gt; 备份数量：默认在HDFS上存放三份，本地一份、同机架内其他节点一份、不同机架节点一份
&gt;


4. Client 请求 3 台 DataNode 中的一台 A 上传数据（本质上是一个 RPC 调用，建立 pipeline ），A 收到请求会继续调用 B，然后 B 调用 C，将整个 pipeline 建立完成， 后逐级返回 client；
5. **Client 开始往 A 上传第一个 block（先从磁盘读取数据放到一个本地内存缓存），以 packet 为单位（默认64K），A 收到一个 packet 就会传给 B，B 传给 C**。A 每传一个 packet 会放入一个应答队列等待应答；
6. 数据被分割成一个个 packet 数据包在 pipeline 上依次传输，在 pipeline 反方向上， 逐个发送 ack（命令正确应答），最终由 pipeline 中第一个 DataNode 节点 A 将 pipelineack 发送给 Client；

7. 当一个 block 传输完成之后，Client 再次请求 NameNode 上传第二个 block，重复步骤 2；

## HDFS文件读取过程

![sfaz8j.png](https://files.catbox.moe/sfaz8j.png)

1. Client远程调用请求NameNode，获取文件块位置列表

2. NameNode会视情况返回文件的部分或者全部block列表；对于每个block，NameNode返回含副本的所有DataNode 地址；计算传输最快最优的DataNode

 &gt;返回的DataNode地址，会按照集群拓扑结构计算客户端的距离，然后进行排序，排序两个规则：网络拓扑结构中距离 Client 近的排靠前；心跳机制中超时汇报的 DN 状态为 STALE，这样的排靠后；

3. Client 选取排序靠前的 DataNode建立输入流，如果客户端本身就是DataNode，那么将从本地直接获取数据(短路读取特性)；

4. 在选定的DataNode上读取该Block的数据

5. 当读完列表的 block 后，若文件读取还没有结束，客户端会继续向NameNode 获取下一批的 block 列表；

6. 读取完一个 block 都会进行 checksum 验证，如果读取 DataNode 时出现错误，客户端会通知 NameNode，然后再从下一个拥有该 block 副本的DataNode 继续读。

7. **read 方法是并行的读取 block 信息，不是一块一块的读取**；NameNode 只是返回Client请求包含块的DataNode地址，并不是返回请求块的数据；

8. 最终读取来所有的 block 会**合并**成一个完整的最终文件。

&gt; 总结：串行写入，数据包先发给节点A，然后节点A发送给B，B在给C
&gt; 并行读取，并行读取block所在的节点，最后合并



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/bigdata/hadoop/a9bc224/  

