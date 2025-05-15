# 5-1. go 操作csv

## 读

```go
//csv文件读取
func ReadCsv(filepath string) {
	//打开文件(只读模式)，创建io.read接口实例
	opencast,err:=os.Open(filepath)
	if err!=nil{
		log.Println(&#34;csv文件打开失败！&#34;)
	}
	defer opencast.Close()
	
	//创建csv读取接口实例
	ReadCsv:=csv.NewReader(opencast)

	//获取一行内容，一般为第一行内容
	read,_:=ReadCsv.Read() //返回切片类型：[chen  hai wei]
	log.Println(read)

	//读取所有内容
	ReadAll,err:=ReadCsv.ReadAll()//返回切片类型：[[s s ds] [a a a]]
	log.Println(ReadAll)

	/*
	  说明：
	   1、读取csv文件返回的内容为切片类型，可以通过遍历的方式使用或Slicer[0]方式获取具体的值。
	   2、同一个函数或线程内，两次调用Read()方法时，第二次调用时得到的值为每二行数据，依此类推。
	   3、大文件时使用逐行读取，小文件直接读取所有然后遍历，两者应用场景不一样，需要注意。
	*/


}
```



## 写

```go

//csv文件写入
func WriterCSV(path string)  {

	//OpenFile读取文件，不存在时则创建，使用追加模式
	File,err:=os.OpenFile(path,os.O_RDWR|os.O_APPEND|os.O_CREATE,0666)
	if err!=nil{
		log.Println(&#34;文件打开失败！&#34;)
	}
	defer File.Close()

	//创建写入接口
	WriterCsv:=csv.NewWriter(File)
    str:=[]string{&#34;chen1&#34;,&#34;hai1&#34;,&#34;wei1&#34;} //需要写入csv的数据，切片类型

	//写入一条数据，传入数据为切片(追加模式)
	err1:=WriterCsv.Write(str)
	if err1!=nil{
		log.Println(&#34;WriterCsv写入文件失败&#34;)
	}
	WriterCsv.Flush() //刷新，不刷新是无法写入的
	log.Println(&#34;数据写入成功...&#34;)
}
```



## 读数据库写入csv

```go
package main

// 从Mysql中导出数据到CSV文件。

import (
	&#34;database/sql&#34;
	&#34;encoding/csv&#34;
	&#34;fmt&#34;
	&#34;os&#34;

	_ &#34;github.com/go-sql-driver/mysql&#34;
)

var (
	tables = []string{&#34;goods&#34;}
	count  = len(tables)
	ch     = make(chan bool, count)
)

func main() {
	db, err := sql.Open(&#34;mysql&#34;, &#34;root:Abcd@123456@tcp(127.0.0.1:3306)/mxshop_goods_srv?charset=utf8&#34;)
	defer db.Close()
	if err != nil {
		panic(err.Error())
	}

	for _, table := range tables {
		go querySQL(db, table, ch)
	}

	for i := 0; i &lt; count; i&#43;&#43; {
		&lt;-ch
	}
	fmt.Println(&#34;Done!&#34;)
}

func querySQL(db *sql.DB, table string, ch chan bool) {
	fmt.Println(&#34;开始处理：&#34;, table)
	rows, _ := db.Query(fmt.Sprintf(&#34;SELECT * from %s&#34;, table))

	columns, err := rows.Columns()
	if err != nil {
		panic(err.Error())
	}

	//values：一行的所有值，长度==列数
	values := make([]sql.RawBytes, len(columns))

	scanArgs := make([]interface{}, len(values))
	for i := range values {
		scanArgs[i] = &amp;values[i]
	}

	var totalValues [][]string
	for rows.Next() {
		var s []string
		err = rows.Scan(scanArgs...) //把每行的内容添加到scanArgs，也添加到了values
		if err != nil {
			panic(err.Error())
		}

		for _, v := range values {
			s = append(s, string(v))
			fmt.Println(s)
		}
		totalValues = append(totalValues, s)
	}

	if err = rows.Err(); err != nil {
		panic(err.Error())
	}
	writeToCSV(table&#43;&#34;.csv&#34;, columns, totalValues)
	ch &lt;- true
}

func writeToCSV(file string, columns []string, totalValues [][]string) {
	// fmt.Println(columns)
	f, err := os.Create(file)
	if err != nil {
		panic(err)
	}
	f.WriteString(&#34;\xEF\xBB\xBF&#34;) //写入UTF-8 格式
	defer f.Close()
	w := csv.NewWriter(f)
	for a, i := range totalValues {
		if a == 0 {
			w.Write(columns)
			w.Write(i)
		} else {
			// fmt.Println(i)
			w.Write(i)
		}
	}
	w.Flush()
	fmt.Println(&#34;处理完毕：&#34;, file)
}

```



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175114/  

