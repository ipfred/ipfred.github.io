# sqoop使用文档


## sqoop介绍

&gt;Sqoop 是一个设计用于在 Hadoop 和关系数据库或大型机之间传输数据的工具。您可以使用 Sqoop 将数据从关系数据库管理系统 (RDBMS)（例如 MySQL 或 Oracle）或大型机导入 Hadoop 分布式文件系统 (HDFS)，在 Hadoop MapReduce 中转换数据，然后将数据导出回 RDBMS 。

Hadoop生态包括： HDFS，Hive，Hbase等
RDBMS体系包括：Mysql、Oracle、DB2等

sqoop是一些工具的集合, 使用的语法如下
```shell
sqoop tool-name [tool-arguments]
```


## 命令工具
可用的命令工具如下
可以使用 --help 查看命令工具的使用方法
`sqoop import --help` `sqoop help import`

| 命令                  | 作用描述                 | 使用方法                                                                 |
| ------------------- | -------------------- | -------------------------------------------------------------------- |
| `codegen`           | 生成与数据库记录交互的代码        | `sqoop codegen --connect &lt;连接字符串&gt; --table &lt;表名&gt;`                       |
| `create-hive-table` | 将表定义导入到 Hive 中       | `sqoop create-hive-table --connect &lt;连接字符串&gt; --table &lt;表名&gt;`             |
| `eval`              | 执行 SQL 语句并显示结果       | `sqoop eval --connect &lt;连接字符串&gt; --query &lt;SQL 语句&gt;`                      |
| **`export`**        | **将 HDFS 目录导出到数据库表** | `sqoop export --connect &lt;连接字符串&gt; --table &lt;表名&gt; --export-dir &lt;HDFS 目录&gt;` |
| `help`              | 列出可用命令               | `sqoop help`                                                         |
| **`import`**        | **将数据库表导入到 HDFS**    | `sqoop import --connect &lt;连接字符串&gt; --table &lt;表名&gt;`                        |
| `import-all-tables` | 将数据库中的所有表导入到 HDFS    | `sqoop import-all-tables --connect &lt;连接字符串&gt;`                          |
| `import-mainframe`  | 将主机服务器上的数据集导入到 HDFS  | `sqoop import-mainframe --connect &lt;连接字符串&gt; --dataset &lt;数据集&gt;`           |
| `job`               | 使用已保存的作业             | `sqoop job --create &lt;作业名&gt; --import --connect &lt;连接字符串&gt; --table &lt;表名&gt;`   |
| `list-databases`    | 列出服务器上的可用数据库         | `sqoop list-databases --connect &lt;连接字符串&gt;`                             |
| `list-tables`       | 列出数据库中的可用表           | `sqoop list-tables --connect &lt;连接字符串&gt;`                                |
| `merge`             | 合并增量导入的结果            | `sqoop merge --new-data &lt;新数据目录&gt; --onto &lt;旧数据目录&gt;`                      |
| `metastore`         | 运行独立的 Sqoop 元数据存储服务  | `sqoop metastore`                                                    |
| `version`           | 显示版本信息               | `sqoop version`                                                      |

## import 

&gt; 功能 将关系型数据表导入到HDFS中，可以设置存储格式为Avro 或者 SequenceFiles

`sqoop import --help` 查看该命令工具的用法

参数可以按功能来分组，包括：Common参数、Hive参数、Import control 参数、HBase 参数、HCatalog 参数、Accumulo 参数、

### common参数

| Argument                             | Description                                                               |
| :----------------------------------- | :------------------------------------------------------------------------ |
| `--connect &lt;jdbc-uri&gt;`               | Specify JDBC connect string                                               |
| `--connection-manager &lt;class-name&gt;`  | Specify connection manager class to use                                   |
| `--driver &lt;class-name&gt;`              | Manually specify JDBC driver class to use                                 |
| `--hadoop-mapred-home &lt;dir&gt;`         | Override $HADOOP_MAPRED_HOME                                              |
| `--help`                             | Print usage instructions                                                  |
| `--password-file`                    | Set path for a file containing the authentication password                |
| `-P`                                 | Read password from console                                                |
| `--password &lt;password&gt;`              | Set authentication password                                               |
| `--username &lt;username&gt;`              | Set authentication username                                               |
| `--verbose`                          | Print more information while working                                      |
| `--connection-param-file &lt;filename&gt;` | Optional properties file that provides connection parameters              |
| `--relaxed-isolation`                | Set connection transaction isolation to read uncommitted for the mappers. |

### 验证参数

可以验证同步之后的表的数据量是否一致

| 参数                                         | 描述                 |
| :----------------------------------------- | :----------------- |
| `--validate`                               | 启用复制数据的验证，仅支持单表复制。 |
| `--validator &lt;class-name&gt;`                 | 指定要使用的验证器类。        |
| `--validation-threshold &lt;class-name&gt;`      | 指定要使用的验证阈值类。       |
| `--validation-failurehandler &lt;class-name&gt;` | 指定要使用的验证失败处理程序类。   |

### import control 参数

| 参数                                | 描述                                                       |
| :-------------------------------- | :------------------------------------------------------- |
| `--append`                        | 将数据附加到 HDFS 中的现有数据集                                      |
| `--as-avrodatafile`               | 将数据导入 Avro 数据文件                                          |
| `--as-sequencefile`               | 将数据导入 SequenceFiles                                      |
| `--as-textfile`                   | 以纯文本形式导入数据（默认）                                           |
| `--as-parquetfile`                | 将数据导入 Parquet 文件                                         |
| `--boundary-query &lt;statement&gt;`    | 用于创建分割的边界查询                                              |
| `--columns &lt;col,col,col…&gt;`        | 从表中导入的列                                                  |
| `--delete-target-dir`             | 如果存在则删除导入目标目录                                            |
| `--direct`                        | 如果数据库存在，则使用直接连接器                                         |
| `--fetch-size &lt;n&gt;`                | 一次从数据库读取的条目数。                                            |
| `--inline-lob-limit &lt;n&gt;`          | 设置内联 LOB 的最大大小                                           |
| `-m,--num-mappers &lt;n&gt;`            | 使用_n个_map任务并行导入                                          |
| `-e,--query &lt;statement&gt;`          | 导入的结果_`statement`_。                                      |
| `--split-by &lt;column-name&gt;`        | 用于分割工作单元的表格列。不能与 `--autoreset-to-one-mapper`选项一起使用。      |
| `--split-limit &lt;n&gt;`               | 每个分割大小的上限。这仅适用于整数和日期列。对于日期或时间戳字段，以秒为单位计算。                |
| `--autoreset-to-one-mapper`       | 如果表没有主键且未提供拆分列，则导入应使用一个映射器。不能与 `--split-by &lt;col&gt;`选项一起使用。 |
| `--table &lt;table-name&gt;`            | 表格阅读                                                     |
| `--target-dir &lt;dir&gt;`              | HDFS 目标目录                                                |
| `--temporary-rootdir &lt;dir&gt;`       | 导入期间创建的临时文件的 HDFS 目录（覆盖默认的“_sqoop”）                      |
| `--warehouse-dir &lt;dir&gt;`           | 表目标的 HDFS 父级                                             |
| `--where &lt;where clause&gt;`          | 导入期间使用的 WHERE 子句                                         |
| `-z,--compress`                   | 启用压缩                                                     |
| `--compression-codec &lt;c&gt;`         | 使用 Hadoop 编解码器（默认 gzip），parquet格式需要加这个配置                 |
| `--null-string &lt;null-string&gt;`     | 为字符串列写入空值的字符串                                            |
| `--null-non-string &lt;null-string&gt;` | 对于非字符串列，要写入空值的字符串                                        |

### 增量导入参数

| 参数                     | 描述                                                                                |
| :--------------------- | :-------------------------------------------------------------------------------- |
| `--check-column (col)` | 指定在确定要导入哪些行时要检查的列。（该列不应为 CHAR/NCHAR/VARCHAR/VARNCHAR/LONGVARCHAR/LONGNVARCHAR 类型） |
| `--incremental (mode)` | 指定 Sqoop 如何确定哪些行是新行。include和 的`mode` 合法 值。 `append``lastmodified`                 |
| `--last-value (value)` | 指定上次导入的检查列的最大值。                                                                   |

### 输出行格式化参数，写入到hdfs中的行数据

| 参数                                | 描述                                               |
| :-------------------------------- | :----------------------------------------------- |
| `--enclosed-by &lt;char&gt;`            | 设置必填字段括字符                                        |
| `--escaped-by &lt;char&gt;`             | 设置转义字符                                           |
| `--fields-terminated-by &lt;char&gt;`   | 设置字段分隔符，hive默认分隔符 是‘\001’, 如果行数据以，分割，需设置这个       |
| `--lines-terminated-by &lt;char&gt;`    | 设置行尾字符                                           |
| `--mysql-delimiters`              | 使用 MySQL 的默认分隔符集：字段：`,` 行：`\n` 转义符：`\` 可选封闭符：`&#39;` |
| `--optionally-enclosed-by &lt;char&gt;` | 设置字段包围字符                                         |

支持的转义字符

- `\b` (backspace)
- `\n` (newline)
- `\r` (carriage return)
- `\t` (tab)
- `\&#34;` (double-quote)
- `\\&#39;` (single-quote)
- `\\` (backslash)
- `\0` (NUL)

### 输入解析参数

| 参数                                      | 描述        |
| :-------------------------------------- | :-------- |
| `--input-enclosed-by &lt;char&gt;`            | 设置必填字段    |
| `--input-escaped-by &lt;char&gt;`             | 设置输入转义字符  |
| `--input-fields-terminated-by &lt;char&gt;`   | 设置输入字段分隔符 |
| `--input-lines-terminated-by &lt;char&gt;`    | 设置输入行尾字符  |
| `--input-optionally-enclosed-by &lt;char&gt;` | 设置字段包围字符  |

### hive参数(常用)

| 参数                           | 描述                                                                                                |
| :--------------------------- | :------------------------------------------------------------------------------------------------ |
| `--hive-home &lt;dir&gt;`          | 覆盖`$HIVE_HOME`                                                                                    |
| `--hive-import`              | 将表导入 Hive（如果未设置分隔符，则使用 Hive 的默认分隔符。）                                                              |
| `--hive-overwrite`           | 覆盖 Hive 表中的现有数据。                                                                                  |
| `--create-hive-table`        | true,目标表存在会报错，默认 false。                                                                           |
| `--hive-table &lt;table-name&gt;`  | 设置导入到 Hive 时使用的表名。                                                                                |
| `--hive-drop-import-delims`  | 导入到 Hive 时从字符串字段中 删除_\n_、_\r_和_\01 。_                                                             |
| `--hive-delims-replacement`  | 导入 Hive 时，将字符串字段中的 _\n_、_\r_和_\01_ 替换为用户定义的字符串。                                                   |
| `--hive-partition-key`       | 要进行分区的 Hive 字段的名称                                                                                 |
| `--hive-partition-value &lt;v&gt;` | 在此作业中，作为导入到 Hive 中的分区键的字符串值。                                                                      |
| `--map-column-hive &lt;map&gt;`    | 覆盖配置列从 SQL 类型到 Hive 类型的默认映射。如果在此参数中指定逗号，请使用 URL 编码的键和值，例如，使用 DECIMAL(1%2C%201) 而不是 DECIMAL(1, 1)。 |
### HBase 参数

```shell
--column-family &lt;family&gt;    设置导入的目标列族
--hbase-create-table    如果指定，则创建缺失的 HBase 表
--hbase-row-key &lt;col&gt;   指定使用哪个输入列作为行键
	如果输入表包含复合
	则 &lt;col&gt; 必须采用
	逗号分隔的复合键列表
	属性
--hbase-table &lt;table-name&gt;  指定要用作目标的 HBase表（而不是 HDFS）
--hbase-bulkload    启用批量加载
```

### 使用例子

#### mysql数据导入到hive，txt格式  分区表
```shell
sqoop import --connect jdbc:mysql://192.168.8.1:3306/{mysql_db}?tinyInt1isBit=false \ 
--username user001 \  
--password passwd001 \  
--query &#34;{query}&#34; \  
--fields-terminated-by &#39;,&#39; \  
--hive-drop-import-delims \  
--null-string &#39;\\\\N&#39;   \  
--null-non-string &#39;\\\\N&#39;  \  
--lines-terminated-by &#34;\\n&#34; \  
--hive-overwrite \  
--hive-import \  
--target-dir /tmp/{hive_table} \  
--delete-target-dir \  
# 分区相关
--hive-partition-key {partition_key} \  
--hive-partition-value {partition_value} \  
--hive-database {hive_db} \  
--hive-table {hive_table} \  
--split-by &#39;{split}&#39; \  
-m {m} \
```

#### mysql导入到hive，parquet格式
```shell

sqoop import -D mapred.job.name=ETL-${table_name} \
  --connect jdbc:mysql://192.168.8.61:3306/CN_Proj_DB?tinyInt1isBit=false  \
 --username user001 \
 --password  passwd001 \
 --query &#34;select * from ${table_name} where 1=1 and \$CONDITIONS&#34; \
 --fields-terminated-by &#39;\001&#39; \
 --hive-drop-import-delims \
 --delete-target-dir  \
 --null-string &#39;\\N&#39;   \
 --null-non-string &#39;\\N&#39;  \
 --lines-terminated-by &#34;\n&#34;  \
 --target-dir /yyq_test/${table_name}  \
 --hive-import \
 --hive-database bigworktest \
 --hive-table ${table_name}  \
 --split-by id \
 -m 3 \
 # 压缩格式 parquet文件
 --compress \
 --compression-codec org.apache.hadoop.io.compress.SnappyCodec \
 --hive-overwrite \
 --as-parquetfile
```

#### mysql导入到hdfs文件
```shell
sqoop import \
--connect jdbc:mysql://192.168.109.1:3306/userdb \
--username root \
--password root \
--delete-target-dir \
--fields-terminated-by &#39;\t&#39;  # 分隔符
--target-dir /sqoopresult \  # 导入的hdfs目录
--table emp 
--m 1
```
#### mysql增量导入

append模式
```shell
sqoop import \
--connect jdbc:mysql://192.168.109.1:3306/userdb \
--username root  --password root \
--table emp --m 1 \
--target-dir /appendresult \
--incremental append \
--check-column id \  # 主键
--last-value  1205  # 追加的id
```
Lastmodified模式 按照时间戳字段
```shell
sqoop import \
--connect jdbc:mysql://192.168.109.1:3306/userdb \
--username root \
--password root \
--table customertest \
--target-dir /lastmodifiedresult \
--check-column last_mod \
--incremental lastmodified \
--last-value &#34;2019-09-03 22:59:45&#34; \
--m 1 \
# --merge-key id 修改旧的，新增新的
--append
```
## import-all-tables

&gt; 将一组表从关系型数据库导入到HDFS中

语法
```shell
sqoop import-all-tables (generic-args) (import-args)
```

每个组的参数参考官方文档

[Sqoop User Guide (v1.4.7) (apache.org)](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_import_all_tables_literal)


## export

&gt; HDFS数据 到 RDBMS

语法
```shell
sqoop export (generic-args) (export-args)
```

### common参数

| 争论                                   | 描述                     |
| :----------------------------------- | :--------------------- |
| `--connect &lt;jdbc-uri&gt;`               | 指定 JDBC 连接字符串          |
| `--connection-manager &lt;class-name&gt;`  | 指定要使用的连接管理器类           |
| `--driver &lt;class-name&gt;`              | 手动指定要使用的 JDBC 驱动程序类    |
| `--hadoop-mapred-home &lt;dir&gt;`         | 覆盖 $HADOOP_MAPRED_HOME |
| `--help`                             | 打印使用说明                 |
| `--password-file`                    | 设置包含身份验证密码的文件路径        |
| `-P`                                 | 从控制台读取密码               |
| `--password &lt;password&gt;`              | 设置认证密码                 |
| `--username &lt;username&gt;`              | 设置认证用户名                |
| `--verbose`                          | 工作时打印更多信息              |
| `--connection-param-file &lt;filename&gt;` | 提供连接参数的可选属性文件          |
| `--relaxed-isolation`                | 将连接事务隔离设置为映射器读取未提交。    |


### 验证参数

| 参数                                         | 描述                 |
| :----------------------------------------- | :----------------- |
| `--validate`                               | 启用复制数据的验证，仅支持单表复制。 |
| `--validator &lt;class-name&gt;`                 | 指定要使用的验证器类。        |
| `--validation-threshold &lt;class-name&gt;`      | 指定要使用的验证阈值类。       |
| `--validation-failurehandler &lt;class-name&gt;` | 指定要使用的验证失败处理程序类。   |


### 导出控制参数

| 参数                                      | 描述                                       |
| :-------------------------------------- | :--------------------------------------- |
| `--columns &lt;col,col,col…&gt;`              | 要导出到表的列                                  |
| `--direct`                              | 使用直接导出快速路径                               |
| `--export-dir &lt;dir&gt;`                    | 导出的 HDFS 源路径                             |
| `-m,--num-mappers &lt;n&gt;`                  | 使用_n个_map任务并行导出                          |
| `--table &lt;table-name&gt;`                  | 要填充的表格                                   |
| `--call &lt;stored-proc-name&gt;`             | 要调用的存储过程                                 |
| `--update-key &lt;col-name&gt;`               | 用于更新的锚点列。如果有多个列，请使用逗号分隔的列表。              |
| `--update-mode &lt;mode&gt;`                  | 指定当数据库中发现具有不匹配键的新行时如何执行更新。               |
|                                         | `updateonly`（默认）和 `allowinsert`。不存在会插入数据 |
| `--input-null-string &lt;null-string&gt;`     | 对于字符串列，将被解释为空的字符串                        |
| `--input-null-non-string &lt;null-string&gt;` | 对于非字符串列，将被解释为空的字符串                       |
| `--staging-table &lt;staging-table-name&gt;`  | 数据在插入目标表之前将暂存于其中的表。                      |
| `--clear-staging-table`                 | 指示可以删除暂存表中的所有数据。                         |
| `--batch`                               | 使用批处理模式执行底层语句。                           |

### 输入解析参数

|争论|描述|
|:--|:--|
|`--input-enclosed-by &lt;char&gt;`|设置必填字段|
|`--input-escaped-by &lt;char&gt;`|设置输入转义字符|
|`--input-fields-terminated-by &lt;char&gt;`|设置输入字段分隔符|
|`--input-lines-terminated-by &lt;char&gt;`|设置输入行尾字符|
|`--input-optionally-enclosed-by &lt;char&gt;`|设置字段包围字符|

  

### 输出行格式化参数

|争论|描述|
|:--|:--|
|`--enclosed-by &lt;char&gt;`|设置必填字段括字符|
|`--escaped-by &lt;char&gt;`|设置转义字符|
|`--fields-terminated-by &lt;char&gt;`|设置字段分隔符|
|`--lines-terminated-by &lt;char&gt;`|设置行尾字符|
|`--mysql-delimiters`|使用 MySQL 的默认分隔符集：字段：`,` 行：`\n` 转义符：`\` 可选封闭符：`&#39;`|
|`--optionally-enclosed-by &lt;char&gt;`|设置字段包围字符|

Sqoop 会自动生成代码来解析和解释包含要导出回数据库的数据的文件记录。如果这些文件是使用非默认分隔符创建的（逗号分隔的字段和换行符分隔的记录），则应再次指定相同的分隔符，以便 Sqoop 可以解析您的文件。


### 代码生成参数

| 争论                      | 描述                                                        |
| :---------------------- | :-------------------------------------------------------- |
| `--bindir &lt;dir&gt;`        | 编译对象的输出目录                                                 |
| `--class-name &lt;name&gt;`   | 设置生成的类名。这将覆盖 `--package-name`。与 结合使用时 `--jar-file`，设置输入类。 |
| `--jar-file &lt;file&gt;`     | 禁用代码生成；使用指定的 jar                                          |
| `--outdir &lt;dir&gt;`        | 生成代码的输出目录                                                 |
| `--package-name &lt;name&gt;` | 将自动生成的类放入此包中                                              |
| `--map-column-java &lt;m&gt;` | 覆盖配置列的从 SQL 类型到 Java 类型的默认映射。                             |

### 使用例子
#### HDFS导出到MySQL

```shell
sqoop export \
--connect jdbc:mysql://192.168.109.1:3306/userdb \
--username root \
--password root \
--table employee \
--export-dir /emp/emp_data
```

#### 更新导出
只更新操作
```shell
sqoop export \
--connect jdbc:mysql://192.168.109.1:3306/userdb \
--username root --password root \
--table updateonly \
--export-dir /updateonly_2/ \
# 更新导出
--update-key id \
--update-mode updateonly  # 只更新操作
```

更新插入模式
```shell
sqoop export \
--connect jdbc:mysql://192.168.8.108:3306/ipv6_locate?serverTimezone=Asia/Shanghai\&amp;useUnicode=true\&amp;characterEncoding=utf8\&amp;autoReconnect=true\&amp;failOverReadOnly=false \ # 有时候可能会出现中文乱码问题，需要添加编码集
--username root \
--password passwd01 \
--table mall_rpt_ip_with_scene \  # 目标mysql中的表名
--update-key &#34;id&#34; \   # 指定目标表的主键
--update-mode allowinsert \  # 指定目标表的更新模式
--input-null-string &#39;\\N&#39; \
--input-null-non-string &#39;\\N&#39; \
--input-fields-terminated-by &#39;\001&#39; \	# hive中字段间的分隔符
--input-lines-terminated-by &#34;\n&#34; \		# hive中行与行之间的分隔符	
--export-dir hdfs://cdh1.host.com:8020/user/hive/warehouse/wsp_test.db/mall_rpt_ip_with_scene \  # hive表存放的位置
--m 1  # 并行度 
```

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: http://localhost:1313/bigdata/hadoop/20240606150541/  

