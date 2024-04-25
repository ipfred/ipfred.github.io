# hadoop命令行的使用

## HDFS 的命令行使用

  

如果没有配置 hadoop 的环境变量，则在 hadoop 的安装目录下的bin目录中执行以下命令，如已配置 hadoop 环境变量，则可在任意目录下执行



- help
```shell
格式:  hdfs dfs -help 操作命令
作用: 查看某一个操作命令的参数信息
```
- ls
```shell
格式：hdfs dfs -ls  URI
作用：类似于Linux的ls命令，显示文件列表
```
- lsr
```shell
格式  :   hdfs  dfs -lsr URI
作用  : 在整个目录下递归执行ls, 与UNIX中的ls-R类似
```
- mkdir
```shell
格式 ： hdfs  dfs  -mkdir [-p] &lt;paths&gt;
作用 :   以&lt;paths&gt;中的URI作为参数，创建目录。使用-p参数可以递归创建目录
```
- put
```shell
格式   ： hdfs dfs -put &lt;localsrc &gt;  ... &lt;dst&gt;
作用 ：  将单个的源文件src或者多个源文件srcs从本地文件系统拷贝到目标文件系统中（&lt;dst&gt;对应的路径）。也可以从标准输入中读取输入，写入目标文件系统中

hdfs dfs -put  /rooot/bigdata.txt  /dir1
```
- moveFromLocal
```shell
格式： hdfs  dfs -moveFromLocal  &lt;localsrc&gt;   &lt;dst&gt;
作用:  和put命令类似，但是源文件localsrc拷贝之后自身被删除

hdfs  dfs -moveFromLocal  /root/bigdata.txt  /
```
- copyFromLocal
```shell
格式:  hdfs dfs -copyFromLocal &lt;localsrc&gt; ... &lt;dst&gt;
作用: 从本地文件系统中拷贝文件到hdfs路径去
```
- appendToFile
```shell
格式: hdfs dfs -appendToFile &lt;localsrc&gt; ... &lt;dst&gt;
作用: 追加一个或者多个文件到hdfs指定文件中.也可以从命令行读取输入.

 hdfs dfs -appendToFile  a.xml b.xml  /big.xml
```
- moveToLocal
```shell
在 hadoop 2.6.4 版本测试还未未实现此方法
格式：hadoop  dfs  -moveToLocal [-crc] &lt;src&gt; &lt;dst&gt;
作用：将本地文件剪切到 HDFS
```
- get
```shell
格式   hdfs dfs  -get [-ignorecrc ]  [-crc]  &lt;src&gt; &lt;localdst&gt;

作用：将文件拷贝到本地文件系统。 CRC 校验失败的文件通过-ignorecrc选项拷贝。 文件和CRC校验可以通过-CRC选项拷贝

hdfs dfs  -get   /bigdata.txt  /export/servers
```
- getmerge
```shell
格式: hdfs dfs -getmerge &lt;src&gt; &lt;localdst&gt;
作用: 合并下载多个文件，比如hdfs的目录 /aaa/下有多个文件:log.1, log.2,log.3,...
```
- copyToLocal
```shell
格式:  hdfs dfs -copyToLocal &lt;src&gt; ... &lt;localdst&gt;
作用:  从hdfs拷贝到本地
```
- mv
```shell
格式  ： hdfs  dfs -mv URI   &lt;dest&gt;
作用： 将hdfs上的文件从原路径移动到目标路径（移动之后文件删除），该命令不能跨文件系统
hdfs  dfs  -mv  /dir1/bigdata.txt   /dir2
```
- rm
```shell
格式： hdfs dfs -rm [-r] 【-skipTrash】 URI 【URI 。。。】
作用： 删除参数指定的文件，参数可以有多个。  此命令只删除文件和非空目录。
如果指定-skipTrash选项，那么在回收站可用的情况下，该选项将跳过回收站而直接删除文件；
否则，在回收站可用时，在HDFS Shell 中执行此命令，会将文件暂时放到回收站中。

hdfs  dfs  -rm  -r  /dir1
```
- cp
```shell
格式: hdfs  dfs  -cp URI [URI ...] &lt;dest&gt;
作用： 将文件拷贝到目标路径中。如果&lt;dest&gt;  为目录的话，可以将多个文件拷贝到该目录下。
-f
选项将覆盖目标，如果它已经存在。
-p
选项将保留文件属性（时间戳、所有权、许可、ACL、XAttr）。

hdfs dfs -cp /dir1/a.txt  /dir2/bigdata.txt
```
- cat
```shell
hdfs dfs  -cat  URI [uri  ...]
作用：将参数所指示的文件内容输出到stdout
hdfs dfs  -cat /bigdata.txt
```
- tail
```shell
格式: hdfs dfs -tail path
作用: 显示一个文件的末尾
```
- text
```shell
格式:hdfs dfs -text path
作用: 以字符形式打印一个文件的内容
```
- chmod
```shell
格式:hdfs  dfs  -chmod  [-R]  URI[URI  ...]
作用：改变文件权限。如果使用  -R 选项，则对整个目录有效递归执行。使用这一命令的用户必须是文件的所属用户，或者超级用户。
hdfs dfs -chmod -R 777 /bigdata.txt
```
- chown
```shell
格式:  hdfs  dfs  -chmod  [-R]  URI[URI ...]
作用：  改变文件的所属用户和用户组。如果使用  -R 选项，则对整个目录有效递归执行。使用这一命令的用户必须是文件的所属用户，或者超级用户。
hdfs  dfs  -chown  -R hadoop:hadoop  /bigdata.txt
```
- df
```shell
格式: hdfs dfs  -df  -h  path
作用: 统计文件系统的可用空间信息
```
- du
```shell
格式: hdfs dfs -du -s -h path
作用: 统计文件夹的大小信息
```
- count
```shell
格式: hdfs dfs -count path
作用: 统计一个指定目录下的文件节点数量
```
- setrep
- expunge (慎用)
```shell
格式:  hdfs dfs -setrep num filePath
作用: 设置hdfs中文件的副本数量
注意: 即使设置的超过了datanode的数量,副本的数量也最多只能和datanode的数量是一致的
```

## hdfs的高级使用命令  
### HDFS文件限额配置  
  
在多人共用HDFS的环境下，配置设置非常重要。特别是在 Hadoop 处理大量资料的环境，如果没有配额管理，很容易把所有的空间用完造成别人无法存取。HDFS 的配额设定是针对目录而不是针对账号，可以让每个账号仅操作某一个目录，然后对目录设置配置。  
  
HDFS 文件的限额配置允许我们以文件个数，或者文件大小来限制我们在某个目录下上传的文件数量或者文件内容总量，以便达到我们类似百度网盘网盘等限制每个用户允许上传的最大的文件的量。  
  
```shell
 hdfs dfs -count -q -h /user/root/dir1  #查看配额信息
```
    

#### 数量限额  
  
```shell
hdfs dfs  -mkdir -p /user/root/dir    #创建hdfs文件夹
hdfs dfsadmin -setQuota 2  dir      # 给该文件夹下面设置最多上传两个文件，发现只能上传一个文件

hdfs dfsadmin -clrQuota /user/root/dir  # 清除文件数量限制
  
```
  

#### 空间大小限额  
  
在设置空间配额时，设置的空间至少是 block_size * 3 大小  

```shell
hdfs dfsadmin -setSpaceQuota 4k /user/root/dir   # 限制空间大小4KB
hdfs dfs -put  /root/a.txt  /user/root/dir
# 生成任意大小的文件
dd if=/dev/zero of=1.txt  bs=1M count=2     #生成2M的文件

# 清除空间配额限制
hdfs dfsadmin -clrSpaceQuota /user/root/dir
```

### HDFS 的安全模式  

安全模式是hadoop的一种保护机制，用于保证集群中的数据块的安全性。当集群启动的时候，会首先进入安全模式。当系统处于安全模式时会检查数据块的完整性。  
  
假设我们设置的副本数（即参数dfs.replication）是3，那么在datanode上就应该有3个副本存在，假设只存在2个副本，那么比例就是2/3=0.666。hdfs默认的副本率0.999。我们的副本率0.666明显小于0.999，因此系统会自动的复制副本到其他dataNode，使得副本率不小于0.999。如果系统中有5个副本，超过我们设定的3个副本，那么系统也会删除多于的2个副本。  
  
在安全模式状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。在，当整个系统达到安全标准时，HDFS自动离开安全模式。30s  
  
安全模式操作命令  
  
```shell
hdfs  dfsadmin  -safemode  get #查看安全模式状态
hdfs  dfsadmin  -safemode  enter #进入安全模式
hdfs  dfsadmin  -safemode  leave #离开安全模式
```
  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/bigdata/hadoop/hadoop%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%9A%84%E4%BD%BF%E7%94%A8/  

