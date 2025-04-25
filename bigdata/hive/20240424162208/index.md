# hive常见的命令

[Hive官方操作手册](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)

## Hive特性

1. hive基于Hadoop，将结构化数据映射为一张数据库表，提供类SQL查询功能
2. Hive**元数据存储**,通常存储在关系形数据库中，Hive中的元数据包括(表的名称，列，分区，是否为外部表等属性，数据所在的路径)
3. Hive支持MapReduce、Tez和Spark 三种计算引擎。
4. **数据格式**。数据格式用户定义，根据三个属性：列分隔符（通常为空格、”\t”、”\x001″）、行分隔符（”\n”）以及读取文件数据的方法（Hive 中默认有三个文件格式 TextFile，SequenceFile 以及 RCFile）
5. **数据更新**。不支持对数据的更新，使用更新可以借助Impala引擎&#43;kudu数据格式
6. **索引**。有hive版本支持建立索引，但是各种问题，不建议使用。
7. Hive中包含的**数据模型**，DB、Table，External Table，Partition，Bucket  
	- db：在hdfs中表现为`hive.metastore.warehouse.dir`目录下一个文件夹。
	- table：在hdfs中表现所属db目录下一个文件夹。
	- external table：与table类似，不过其数据存放位置可以在任意指定路径。
	- partition：在hdfs中表现为table目录下的子目录。
	- bucket：在hdfs中表现为同一个表目录下根据hash散列之后的多个文件。

## Hive表类型
### hive数据类型

- Hive的基本数据类型有：`TINYINT，SAMLLINT，INT，BIGINT，BOOLEAN，FLOAT，DOUBLE，STRING，TIMESTAMP(V0.8.0&#43;)和BINARY(V0.8.0&#43;)`。
- Hive的集合类型有：`STRUCT，MAP和ARRAY`。
- Hive表：内部表、外部表、分区表和桶表。  
- 表的元数据保存传统的数据库的表中，**当前hive只支持Derby和MySQL数据库**。

### Hive内部表

Hive中的内部表和传统数据库中的表在概念上是类似的，Hive的每个表都有自己的存储目录，除了外部表外，所有的表数据都存放在配置在`hive-site.xml`文件的`${hive.metastore.warehouse.dir}/table_name`目录下。

```sql
-- 创建内部表
CREATE TABLE IF NOT EXISTS students(user_no INT,name STRING,sex STRING,  
         grade STRING COMMOT &#39;班级&#39;）COMMONT &#39;学生表&#39;  
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY &#39;,&#39;
STORE AS TEXTFILE;
```
### Hive外部表

被external修饰的为外部表（external table），外部表指向已经存在在Hadoop HDFS上的数据，除了在删除外部表时只删除元数据而不会删除表数据外，其他和内部表很像

```sql
-- 创建外部表
CREATE EXTERNAL TABLE IF NOT EXISTS students(user_no INT,name STRING,sex STRING,  
         class STRING COMMOT &#39;班级&#39;）COMMONT &#39;学生表&#39;  
ROW FORMAT DELIMITED  
FIELDS TERMINATED BY &#39;,&#39;  
STORE AS SEQUENCEFILE 
LOCATION &#39;/usr/test/data/students.txt&#39;;
```
### Hive分区表

分区表的每一个分区都对应数据库中相应分区列的一个索引，分区表中的每一个分区对应一个目录文件，即同一个分区的数据存放一个目录下面，所以分区表如果很大，指定分区会速度快很多

比如说，分区表partitinTable有包含nation(国家)、ds(日期)和city(城市)3个分区，其中nation = china，ds = 20130506，city = Shanghai则对应HDFS上的目录为：
`/datawarehouse/partitinTable/nation=china/city=Shanghai/ds=20130506/`。

```sql
-- 创建分区表
CREATE TABLE IF NOT EXISTS students(user_no INT,name STRING,sex STRING,
         class STRING COMMOT &#39;班级&#39;）COMMONT &#39;学生表&#39;  
PARTITIONED BY (ds STRING,country STRING)  -- 分区的列不属于建表的字段中
ROW FORMAT DELIMITED  --分隔符
FIELDS TERMINATED BY &#39;,&#39;  -- 结束换行符
STORE AS SEQUENCEFILE -- 数据存储格式
;
```
### Hive分桶表

对指定列进行HASH，根据hash值进行切分数据，不同hash结果的数据写到每个桶对应的文件目录中

```sql
-- 创建分桶表
CREATE TABLE IF NOT EXISTS students(user_no INT,name STRING,sex STRING,  
         class STRING COMMOT &#39;班级&#39;,score SMALLINT COMMOT &#39;总分&#39;）COMMONT &#39;学生表&#39;  
PARTITIONED BY (ds STRING,country STRING)  
CLUSTERED BY(user_no) SORTED BY(score) INTO 32 BUCKETS  -- 分桶
ROW FORMAT DELIMITED  
FIELDS TERMINATED BY &#39;,&#39;  
STORE AS SEQUENCEFILE;
```
### Hive视图

逻辑数据结构，将查询结果作为视图，简化查询操作

```sql
CREATE VIEW employee_skills
 AS
SELECT name, skills_score[&#39;DB&#39;] AS DB,
skills_score[&#39;Perl&#39;] AS Perl, 
skills_score[&#39;Python&#39;] AS Python,
skills_score[&#39;Sales&#39;] as Sales, 
skills_score[&#39;HR&#39;] as HR 
FROM employee;
```

### 总结

表的创建官方标准
[LanguageManual DDL -创建表的语法结构](https://cwiki.apache.org/confluence/display/Hive/LanguageManual&#43;DDL#LanguageManualDDL-CreateTable)

```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [column_constraint_specification] [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY &#39;storage.handler.class.name&#39; [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)

```

```sql
-- 用另外一个表的结构来创建
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
LIKE existing_table_or_view_name
[LOCATION hdfs_path];
```

删除表

```sql
DROP TABLE [IF EXISTS] table_name [PURGE];     -- (Note: PURGE available in Hive 0.14.0 and later)
```
清空表
```sql
TRUNCATE [TABLE] table_name [PARTITION partition_spec];
partition_spec:
  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)
```

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: http://localhost:1313/bigdata/hive/20240424162208/  

