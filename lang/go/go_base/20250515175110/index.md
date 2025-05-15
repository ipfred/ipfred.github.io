# 5-0. 文件操作

- 文件操作设计的I/O操作库
  - io -- 提供基本的接口
  - io/ioutil -- 封装了一些常用的接口
  - fmt -- 实现格式化IO. 类似c语言中的printf和scanf
  - bufio -- 实现缓存I/O

# io/ioutil

- 读取文件目录 `ReadDir()`, 返回一个列表`[]os.FileInfo`, 列表中包含指定目录的目录信息

  ```golang
  type FileInfo interface{
    Name() string // 文件名称
    Size() int64  // 文件大小
    Mode() FileMode  //打开模式
    ModTime() time.Time  //文件修改时间
    IsDIr() bool   // 是否是目录
    Sys()  interface{}  // 基础数据源
  }
  ```

  - demo : 查看文件夹中所有文件, 并输出文件名

    ```go
    package main
    
    import (
    	&#34;fmt&#34;
    	&#34;io/ioutil&#34;
    )
    
    func getFileName(filePath string)  {
    	dir, err := ioutil.ReadDir(filePath)
    	if err != nil {
    		fmt.Println(err)
    	}
    	for _,file := range dir{
    		fileName := file.Name()
    		fmt.Println(fileName)
    	}
    }
    
    
    func main() {
    	filePath := &#34;../../&#34;
    	getFileName(filePath)
    
    }
    ```

    

# path/filepath

- Walk() 获取目录下所有文件和文件夹

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;io/fs&#34;
  	&#34;path/filepath&#34;
  )
  
  //walk() 获取指定目录下的所有文件和文件夹
  func WalkDir(dirPath string)   {
  	err := filepath.Walk(dirPath, func(path string, info fs.FileInfo, err error) error {
  		if info ==nil{
  			return err
  		}
  		if info.IsDir(){
  			return nil
  		}
  		fmt.Println(path)
  		return nil
  	})
  	if err != nil{
  		fmt.Printf(&#34;filepath.Walk() returned %v\n&#34;,err)
  	}
  }
  
  func main() {
  	WalkDir(&#34;./&#34;)
  }
  
  ```


# OS

- os.Makedir创建文件夹,需要指定权限
  - 创建文件夹没有权限会报错
- os.MkdirAll创建文件夹 , 需要制定权限
  - 创建文件夹, 不存在就创建, 存在返回nil
- os.Remove() 删除指定文件夹, 非空不能删除
- os.RemoveAll() 删除指定文件夹, 非空也删除
- os.OpenFile() **创建/读取文件**
- os.Open() 读文件
- os.ModePerm  返回值:  `-rwxrwxrwx`,类似0777

&gt; ###### os.Open 与读的方式
&gt;
&gt; f ,e := os.OpenFile(&#34;data.csv&#34;)
&gt;
&gt; ###### os.OpenFile 读写方式
&gt;
&gt; f ,e := os.OpenFile(&#34;data.csv&#34;, os.O_CREATE|os.O_RDWR, 0644)

### FileInfo

**FileInfo** 接口中定义了 File 信息相关的方法。

```go
type FileInfo interface {
    Name() string       // base name of the file 文件名.扩展名 aa.txt
    Size() int64        // 文件大小，字节数 12540
    Mode() FileMode     // 文件权限 -rw-rw-rw-
    ModTime() time.Time // 修改时间 2020-09-23 16:30:53 &#43;0800 CST
    IsDir() bool        // 是否文件夹
    Sys() interface{}   // 基础数据源接口(can return nil)
}
```

### 权限

至于操作权限 perm，除非创建文件时才需要指定，不需要创建新文件时可以将其设定为０。虽然 Golang 语言给 perm 权限设定了很多的常量，但是习惯上也可以直接使用数字，如 0666 (具体含义和 Unix 系统的一致)。
 权限控制：



```go
linux 下有2种文件权限表示方式，即“符号表示”和“八进制表示”。

（1）符号表示方式:
-      ---         ---        ---
type   owner       group      others
文件的权限是这样子分配的 读 写 可执行 分别对应的是 r w x 如果没有那一个权限，用 - 代替
(-文件 d目录 |连接符号)
例如：-rwxr-xr-x

（2）八进制表示方式： 
r ——&gt; 004
w ——&gt; 002
x ——&gt; 001
- ——&gt; 000

0755
0777
0555
0444
0666
```

### 创建文件夹

- os.MkDir(&#34;dirName&#34;,os.ModePerm )，创建一层
- os.MkDirAll(&#34;dirName&#34;,os.ModePerm )，可以创建多层

### 创建文件

- os.Create()，创建文件
  -  采用模式0666（任何人都可读写，不可执行）创建一个名为name的文件，如果文件已存在会替换它（为空文件）
           

### 打开/关闭文件

- os.Open(fileName) 只读

- os.OpenFile(filename,mode,perm)   --文件名称|文件的打开方式|文件的权限

  ```go
  file4,err := os.OpenFile(fileName1,os.O_RDONLY|os.O_WRONLY,os.ModePerm)
  ```

  1. 第一个参数：文件名称    

  2. 第二个参数：文件的打开方式        

     | 模式        | 含义     |
     | ----------- | -------- |
     | os.O_WRONLY | 只写     |
     | os.O_CREATE | 创建文件 |
     | os.O_RDONLY | 只读     |
     | os.O_RDWR   | 读写     |
     | os.O_TRUNC  | 清空     |
     | os.O_APPEND | 追加     |

     ```go
     const (    
         
     // Exactly one of O_RDONLY, O_WRONLY, or O_RDWR must be specified.
             O_RDONLY int = syscall.O_RDONLY // open the file read-only.
             O_WRONLY int = syscall.O_WRONLY // open the file write-only.
             O_RDWR   int = syscall.O_RDWR   // open the file read-write.
             // The remaining values may be or&#39;ed in to control behavior.
             O_APPEND int = syscall.O_APPEND // append data to the file when writing.
             O_CREATE int = syscall.O_CREAT  // create a new file if none exists.
             O_EXCL   int = syscall.O_EXCL   // used with O_CREATE, file must not exist.
             O_SYNC   int = syscall.O_SYNC   // open for synchronous I/O.
             O_TRUNC  int = syscall.O_TRUNC  // truncate regular writable file when opened.
     
     ) 
     
     ```

     

  3. 第三个参数：文件的权限：文件不存在创建文件，需要指定权限

  ```go
  package main
  
  import (
  	// &#34;fmt&#34;
  	&#34;fmt&#34;
  	&#34;os&#34;
  )
  
  func main() {
  	file, err := os.Open(&#34;word.txt&#34;) // 读取文件
  	if err != nil {
  		fmt.Println(&#34;error:&#34;, err)
  		return
  	}
  	defer file.Close()
  	fileinfo, err := file.Stat()
  	if err != nil {
  		fmt.Println(err)
  		return
  	}
  	fileSize := fileinfo.Size()   // 文件大小
  	buffer := make([]byte, fileSize)
  	bytesread, err := file.Read(buffer)  //获取文件内容
  	if err != nil {
  		fmt.Println(&#34;err::&#34;, err)
  	}
  	fmt.Println(&#34;bytes read: &#34;, bytesread)
  	fmt.Println(&#34;bytestream to string: &#34;, string(buffer))
  
  }
  ```


- file.Close() 关闭文件



### 创建/删除文件/目录



- os.Create()，创建文件
  - 采用模式0666（任何人都可读写，不可执行）创建一个名为name的文件，如果文件已存在会替换它（为空文件）
- 删除文件 os.Remove()
- 删除文件夹 os.RemoveAll()

```go
//删除文件
err :=  os.Remove(&#34;/Users/ruby/Documents/pro/a/aa.txt&#34;)
if err != nil{
  fmt.Println(&#34;err:&#34;,err)
  return
}
fmt.Println(&#34;删除文件成功。。&#34;)
//删除目录
err :=  os.RemoveAll(&#34;/Users/ruby/Documents/pro/a/cc&#34;)
if err != nil{
    fmt.Println(&#34;err:&#34;,err)
    return
}
fmt.Println(&#34;删除目录成功。。&#34;)
```

### 判断文件是否存在

Golang 判断文件或文件夹是否存在的方法为使用 os.Stat() 函数返回的错误值进行判断。

1. 如果返回的错误为 nil，说明文件或文件夹存在
2. 如果返回的错误类型使用 **os.IsNotExist()** 判断为 true，说明文件或文件夹不存在

```go
package main
import (
    &#34;log&#34;
    &#34;os&#34;
)

func main() {
    fileInfo,err:=os.Stat(&#34;/Users/ruby/Documents/pro/a/aa.txt&#34;)
    if err!=nil{
        if os.IsNotExist(err){
            log.Fatalln(&#34;file does not exist&#34;)
        }
    }
    log.Println(&#34;file does exist. file information:&#34;)
    log.Println(fileInfo)
}
```

### **读取文件内容**

- 读文件方式一：利用**ioutil.ReadFile**直接从文件读取到[]byte中

```go
func Read0()  (string){
    f, err := ioutil.ReadFile(&#34;file/test&#34;)
    if err != nil {
        fmt.Println(&#34;read fail&#34;, err)
    }
    return string(f)
}
```

- 读文件方式二：先从文件读取到file中，在从file读取到buf, buf在追加到最终的[]byte

```go
func Read1()  (string){
    //获得一个file
    f, err := os.Open(&#34;file/test&#34;)
    if err != nil {
        fmt.Println(&#34;read fail&#34;)
        return &#34;&#34;
    }

    //把file读取到缓冲区中
    defer f.Close()
    var chunk []byte
    buf := make([]byte, 1024)

    for {
        //从file读取到buf中
        n, err := f.Read(buf)
        if err != nil &amp;&amp; err != io.EOF{
            fmt.Println(&#34;read buf fail&#34;, err)
            return &#34;&#34;
        }
        //说明读取结束
        if n == 0 {
            break
        }
        //读取到最终的缓冲区中
        chunk = append(chunk, buf[:n]...)
    }

    return string(chunk)
    //fmt.Println(string(chunk))
}
```

- 读文件方式三：先从文件读取到file, 在从file读取到Reader中，从Reader读取到buf, buf最终追加到[]byte



```go
//先从文件读取到file, 在从file读取到Reader中，从Reader读取到buf, buf最终追加到[]byte，这个排第三
func Read2() (string) {
    fi, err := os.Open(&#34;file/test&#34;)
    if err != nil {
        panic(err)
    }
    defer fi.Close()

    r := bufio.NewReader(fi)
    var chunks []byte

    buf := make([]byte, 1024)

    for {
        n, err := r.Read(buf)
        if err != nil &amp;&amp; err != io.EOF {
            panic(err)
        }
        if 0 == n {
            break
        }
        //fmt.Println(string(buf))
        chunks = append(chunks, buf...)
    }
    return string(chunks)
    //fmt.Println(string(chunks))
}
```

- 读文件方式四：读取到file中，再利用ioutil将file直接读取到[]byte中

```go
//读取到file中，再利用ioutil将file直接读取到[]byte中, 这是最优
func Read3()  (string){
    f, err := os.Open(&#34;file/test&#34;)
    if err != nil {
        fmt.Println(&#34;read file fail&#34;, err)
        return &#34;&#34;
    }
    defer f.Close()

    fd, err := ioutil.ReadAll(f)
    if err != nil {
        fmt.Println(&#34;read to fd fail&#34;, err)
        return &#34;&#34;
    }

    return string(fd)
}
```

四种方式读的速度排名是：前者为优
 方式四 &gt; 方式一 &gt; 方式三 &gt; 方式二

### **写文件**内容

- 写文件方式一：使用 **io.WriteString** 写入文件



```go
func Write0()  {
    fileName := &#34;file/test1&#34;
    strTest := &#34;测试测试&#34;

    var f *os.File
    var err error

    if CheckFileExist(fileName) {  //文件存在
        f, err = os.OpenFile(fileName, os.O_APPEND, 0666) //打开文件
        if err != nil{
            fmt.Println(&#34;file open fail&#34;, err)
            return
        }
    }else {  //文件不存在
        f, err = os.Create(fileName) //创建文件
        if err != nil {
            fmt.Println(&#34;file create fail&#34;)
            return
        }
    }
    //将文件写进去
    n, err1 := io.WriteString(f, strTest)
    if err1 != nil {
        fmt.Println(&#34;write error&#34;, err1)
        return
    }
    fmt.Println(&#34;写入的字节数是：&#34;, n)
}
```

- 写文件方式二：使用 ioutil.WriteFile 写入文件



```go
func Write1()  {
    fileName := &#34;file/test2&#34;
    strTest := &#34;测试测试&#34;
    var d = []byte(strTest)
    err := ioutil.WriteFile(fileName, d, 0666)
    if err != nil {
        fmt.Println(&#34;write fail&#34;)
    }
    fmt.Println(&#34;write success&#34;)
}
```

- 写文件方式三：使用 File(Write,WriteString) 写入文件



```go
func Write2()  {

    fileName := &#34;file/test3&#34;
    strTest := &#34;测试测试&#34;
    var d1 = []byte(strTest)

    f, err3 := os.Create(fileName) //创建文件
    if err3 != nil{
        fmt.Println(&#34;create file fail&#34;)
    }
    defer f.Close()
    n2, err3 := f.Write(d1) //写入文件(字节数组)

    fmt.Printf(&#34;写入 %d 个字节n&#34;, n2)
    n3, err3 := f.WriteString(&#34;writesn&#34;) //写入文件(字节数组)
    fmt.Printf(&#34;写入 %d 个字节n&#34;, n3)
    f.Sync()
}
// 或者
func WriteFile(fPath string) {
	//f, _ := os.OpenFile(fPath,os.O_APPEND,0666)
	f, err := os.OpenFile(fPath, os.O_CREATE|os.O_WRONLY, os.ModeAppend)
	defer f.Close()
	writeString, err := f.WriteString(&#34;你好&#34;)
	if err != nil {
		fmt.Println(writeString)
		panic(err)
		return
	}
	fmt.Println(writeString)
}
```

- 写文件方式四：使用 bufio.NewWriter 写入文件



```go
func Write3()  {
    fileName := &#34;file/test3&#34;
    f, err3 := os.Create(fileName) //创建文件
    if err3 != nil{
        fmt.Println(&#34;create file fail&#34;)
    }
    w := bufio.NewWriter(f) //创建新的 Writer 对象
    n4, err3 := w.WriteString(&#34;bufferedn&#34;)
    fmt.Printf(&#34;写入 %d 个字节n&#34;, n4)
    w.Flush()
    f.Close()
}
```



# JSON文件操作

### 编码JSON

- 编码json: 从其他类型编码成json字符串
- encoding/json
  - **Marshal** 返回interface{}类型 的json编码, 通常interface{}类型会使用map或者结构体
  - **MarshalIndent**  类似于Marshal, 会使用缩进输出格式化json

- 使用map创建json

  ```go
  package main
  
  import (
  	&#34;encoding/json&#34;
  	&#34;fmt&#34;
  )
  
  /**
  map创建json
   */
  func main() {
  	m:=make(map[string]interface{},3)
  	m[&#34;name&#34;] = &#34;evan&#34;
  	m[&#34;age&#34;] = 27
  	m[&#34;language&#34;] = []string{&#34;python&#34;,&#34;golang&#34;,&#34;javascript&#34;}
  
  	result,_:=json.Marshal(m)
  	resultMarshalIndent,_:=json.MarshalIndent(m,&#34;&#34;,&#34;    &#34;)
  	fmt.Println(string(result))
  	fmt.Println(string(resultMarshalIndent))
  }
  
  /**
  {&#34;age&#34;:27,&#34;language&#34;:[&#34;python&#34;,&#34;golang&#34;,&#34;javascript&#34;],&#34;name&#34;:&#34;evan&#34;}
  {
      &#34;age&#34;: 27,
      &#34;language&#34;: [
          &#34;python&#34;,
          &#34;golang&#34;,
          &#34;javascript&#34;
      ],
      &#34;name&#34;: &#34;evan&#34;
  }
  */
  ```

  

- 使用结构体创建json(常用)

  - 当在定义struct 的时, 可以在后面添加标签来控制编码和解码的过程; 是否要编码或者解码某个字段, JSON中的名字字段名称是什么, 可以选择的控制字段有三种
    1. &#34;-&#34; :表示不要解析该字段
    2. &#34;omitempty&#34; : 当字段为空的时候不要解析该字段, 比如: false 0 nil 长度为0的array map slice string
    3. &#34;FieldName&#34; : 当解析JSON的时候使用这个名字

  ```go
  package main
  
  import (
  	&#34;encoding/json&#34;
  	&#34;fmt&#34;
  )
  
  type Person struct {
  	Name     string `json:&#34;name&#34;`
  	Age      int    `json:&#34;age&#34;`
  	Sex      bool   `json:&#34;sex&#34;`
  	Birthday string `json:&#34;birthday&#34;`
  	Company  string `json:&#34;company,omitempty&#34;`
  	Language []string `json:&#34;language&#34;`
  }
  
  func main() {
  	person :=Person{&#34;Evan&#34;,28,true,&#34;1994&#34;,&#34;DCITS&#34;,[]string{
  		&#34;python&#34;,&#34;golang&#34;,
  	}}
  
  	//result,err :=json.Marshal(person)
  	result,err :=json.MarshalIndent(person,&#34;&#34;,&#34;    &#34;)
  	if err !=nil{
  		fmt.Println(err)
  		return
  
  	}
  	fmt.Println(string(result))
  
  }
  /**
  {
      &#34;name&#34;: &#34;Evan&#34;,
      &#34;age&#34;: 28,
      &#34;sex&#34;: true,
      &#34;birthday&#34;: &#34;1994&#34;,
      &#34;company&#34;: &#34;DCITS&#34;,
      &#34;language&#34;: [
          &#34;python&#34;,
          &#34;golang&#34;
      ]
  }
  */
  ```

### 解析JSON

- Unmarshal() 解析json,是Marshal()的反向操作

- 要将json数据解码写入一个接口类型值, 函数会将解码数据为如下对照关系写入接口

  | interface类型 | json类型   |
  | ------------- | ---------- |
  | Bool          | 布尔类型   |
  | Float64       | 数字类型   |
  | string        | 字符串类型 |
  | []interface{} | 数组       |
  | map           | 对象       |
  | nil           | null       |

  - 如果json的值不满足或者不匹配给出的目标类型, 或者JSON数字写入目标类型时溢出, unmarsha()函数会跳过该字段, 尽可能满足其余的解码操作

- struce 解码json(推荐使用)

  ```go
  package main
  
  import (
  	&#34;encoding/json&#34;
  	&#34;fmt&#34;
  )
  
  type Person struct {
  	Name     string   `json:&#34;name&#34;`
  	Age      int      `json:&#34;age&#34;`
  	Sex      bool     `json:&#34;sex&#34;`
  	Birthday string   `json:&#34;birthday&#34;`
  	Company  string   `json:&#34;company,omitempty&#34;`
  	Language []string `json:&#34;language&#34;`
  }
  
  func main() {
  	jsonStr := `
  		{   &#34;name&#34;:&#34;evan&#34;,	
  			&#34;age&#34;:28,
  			&#34;sex&#34;:true,
  			&#34;birthday&#34;:&#34;1994&#34;,
  			&#34;company&#34;:&#34;DCITS&#34;,
  			&#34;language&#34;:[&#34;python&#34;,&#34;golang&#34;,&#34;javascript&#34;]
  		}
  		`
  	var person Person
  
  	err:=json.Unmarshal([]byte(jsonStr),&amp;person)
  
  	if err!=nil {
  		fmt.Println(err)
  		return
  
  	}
  	fmt.Println(person)
  }
  // {evan 28 true 1994 DCITS [python golang javascript]}
  ```

### 拓展 fastjson

- 速度是原生json操作速度的10倍
- 地址: https://github.com/valyala/fastjson

# XML文件操作

- XML是一门标记语言

- XML对大小写敏感

  - Xml 文件

    ```xml
    &lt;?xml version=&#34;1.0&#34; encoding=&#34;utf-8&#34; ?&gt;
    &lt;Wrapper&gt;
        &lt;Note&gt;
            &lt;To&gt;George&lt;/To&gt;
            &lt;From&gt;John&lt;/From&gt;
            &lt;Body&gt;
                &lt;Bodychild&gt;bc1&lt;/Bodychild&gt;
                &lt;Bodychild&gt;bc2&lt;/Bodychild&gt;
            &lt;/Body&gt;
        &lt;/Note&gt;
        &lt;Note&gt;
            &lt;To&gt;George2&lt;/To&gt;
            &lt;From&gt;John2&lt;/From&gt;
            &lt;Body&gt;
                &lt;Bodychild&gt;bc1&lt;/Bodychild&gt;
                &lt;Bodychild&gt;bc2&lt;/Bodychild&gt;
            &lt;/Body&gt;
        &lt;/Note&gt;
    &lt;/Wrapper&gt;
    ```

    

  - golang code 

    ```go
    package main
    
    import (
    	&#34;encoding/xml&#34;
    	&#34;fmt&#34;
    	&#34;io/ioutil&#34;
    )
    
    type Wrapper struct {
    	Note []Note
    }
    
    type Note struct{
    	To string
    	From string
    	Body Body
    }
    
    type Body struct {
    	Bodychild []string
    }
    
    func main() {
    	var res Wrapper
    	content,err:=ioutil.ReadFile(&#34;test.xml&#34;)
    
    	if err!=nil{
    		fmt.Println(err)
    		return
    
    	}
    	err = xml.Unmarshal(content,&amp;res)
    	if err !=nil {
    		fmt.Println(err)
    		return
    	}
    	fmt.Println(&#34;解析后的xml文件为:&#34;)
    	fmt.Println(res)
    }
    ```

    

# 按行读文件

- 方法一

  ```go
  package main
  
  import (
  	&#34;bufio&#34;
  	&#34;fmt&#34;
  	&#34;io&#34;
  	&#34;os&#34;
  )
  
  const FilePath = &#34;./test.txt&#34;
  
  func readLineOfOne() {
  	f, err := os.Open(FilePath)
  	if err!=nil {
  		fmt.Println(err)
  		os.Exit(2)
  	}
  	defer f.Close()
  	bufReader := bufio.NewReader(f)
  	i := 0
  	for {
  		i&#43;&#43;
  		//fmt.Println(i)
  		line,err:= bufReader.ReadBytes(&#39;\n&#39;)
  		fmt.Println(&#34;-&gt;&#34;,string(line))
  		if err== io.EOF {
  			fmt.Println(&#34;read file finished.&#34;)
  			break
  		}else if err!= nil	{
  			fmt.Println(err)
  			os.Exit(2)
  		}
  
  	}
  
  }
  
  func main() {
  	readLineOfOne()
  }
  
  ```

  - bufReader.ReadBytes(&#39;\n&#39;)和 bufReader.ReadString(&#39;\n&#39;)在读到文件最后一行时，会同时返回内容line和io.EOF

# bufio.Reader

- 方法一(一行一行读)

```go
f, err := os.Open(&#34;./src/day1/file_read/1.txt&#34;)
    if err != nil{
        fmt.Println(err)
        os.Exit(2)
    }
    defer f.Close()
    bufReader := bufio.NewReader(f)
    var i = 0
    for{
        i&#43;&#43;
        fmt.Println(i)
        line,err := bufReader.ReadBytes(&#39;\n&#39;)
        fmt.Println(string(line))
        if err == io.EOF {
            fmt.Println(&#34;read the file finished&#34;)
            break
        }
        if err != nil{
            fmt.Println(err)
            os.Exit(2)
        }

    }
```

- 方法二

```go
f, err := os.Open(&#34;./src/day1/file_read/1.txt&#34;)
    if err != nil{
        fmt.Println(err)
        os.Exit(2)
    }
    defer f.Close()
    bufReader := bufio.NewReader(f)
    var buf [256]byte
    var i = 0
    for{
                i&#43;&#43;
        fmt.Println(i)
        n,err := bufReader.Read(buf[:])
        if err == io.EOF {
            fmt.Println(&#34;read the file finished&#34;)
            break
        }
        if err != nil{
            fmt.Println(err)
            os.Exit(2)
        }
        fmt.Println(string(buf[:n]))
    }
```

- bufReader.ReadBytes(&#39;\n&#39;)和 bufReader.ReadString(&#39;\n&#39;)在读到文件最后一行时，会同时返回内容line和io.EOF。而bufReader.Read()读取到末尾时，会先返回内容，然后再下一次迭代时才返回io.EOF

# 读写文件的四种方式

## 读文件

读取的文件放在file/test：也就是file包下的test这个文件，里面写多一点文件

- 读文件方式一：利用ioutil.ReadFile直接从文件读取到[]byte中

- **ioutil.ReadFile** 将文件全部读到内存

```go
func Read0()  (string){
    f, err := ioutil.ReadFile(&#34;file/test&#34;)
    if err != nil {
        fmt.Println(&#34;read fail&#34;, err)
    }
    return string(f)
}
```

- 读文件方式二：先从文件读取到file中，在从file读取到buf, buf在追加到最终的[]byte

- **os.Open** 文件流的方式读取文件

```go
func Read1()  (string){
    //获得一个file
    f, err := os.Open(&#34;file/test&#34;)
    if err != nil {
        fmt.Println(&#34;read fail&#34;)
        return &#34;&#34;
    }

    //把file读取到缓冲区中
    defer f.Close()
    var chunk []byte
    buf := make([]byte, 1024)

    for {
        //从file读取到buf中
        n, err := f.Read(buf)
        if err != nil &amp;&amp; err != io.EOF{
            fmt.Println(&#34;read buf fail&#34;, err)
            return &#34;&#34;
        }
        //说明读取结束
        if n == 0 {
            break
        }
        //读取到最终的缓冲区中
        chunk = append(chunk, buf[:n]...)
    }

    return string(chunk)
    //fmt.Println(string(chunk))
}
```

- 读文件方式三：先从文件读取到file, 在从file读取到Reader中，从Reader读取到buf, buf最终追加到[]byte



```go
//先从文件读取到file, 在从file读取到Reader中，从Reader读取到buf, buf最终追加到[]byte，这个排第三
func Read2() (string) {
    fi, err := os.Open(&#34;file/test&#34;)
    if err != nil {
        panic(err)
    }
    defer fi.Close()

    r := bufio.NewReader(fi)
    var chunks []byte

    buf := make([]byte, 1024)

    for {
        n, err := r.Read(buf)
        if err != nil &amp;&amp; err != io.EOF {
            panic(err)
        }
        if 0 == n {
            break
        }
        //fmt.Println(string(buf))
        chunks = append(chunks, buf...)
    }
    return string(chunks)
    //fmt.Println(string(chunks))
}
```

- 读文件方式四：读取到file中，再利用ioutil将file直接读取到[]byte中



```go
//读取到file中，再利用ioutil将file直接读取到[]byte中, 这是最优
func Read3()  (string){
    f, err := os.Open(&#34;file/test&#34;)
    if err != nil {
        fmt.Println(&#34;read file fail&#34;, err)
        return &#34;&#34;
    }
    defer f.Close()

    fd, err := ioutil.ReadAll(f)
    if err != nil {
        fmt.Println(&#34;read to fd fail&#34;, err)
        return &#34;&#34;
    }

    return string(fd)
}
```

- **读取速度比较**

经过我的测试，这四种方式读的速度排名是：前者为优
 方式四 &gt; 方式一 &gt; 方式三 &gt; 方式二

## 写文件

- 写文件方式一：使用 io.WriteString 写入文件



```go
func Write0()  {
    fileName := &#34;file/test1&#34;
    strTest := &#34;测试测试&#34;

    var f *os.File
    var err error

    if CheckFileExist(fileName) {  //文件存在
        f, err = os.OpenFile(fileName, os.O_APPEND, 0666) //打开文件
        if err != nil{
            fmt.Println(&#34;file open fail&#34;, err)
            return
        }
    }else {  //文件不存在
        f, err = os.Create(fileName) //创建文件
        if err != nil {
            fmt.Println(&#34;file create fail&#34;)
            return
        }
    }
    //将文件写进去
    n, err1 := io.WriteString(f, strTest)
    if err1 != nil {
        fmt.Println(&#34;write error&#34;, err1)
        return
    }
    fmt.Println(&#34;写入的字节数是：&#34;, n)
}
```

- 写文件方式二：使用 ioutil.WriteFile 写入文件



```css
func Write1()  {
    fileName := &#34;file/test2&#34;
    strTest := &#34;测试测试&#34;
    var d = []byte(strTest)
    err := ioutil.WriteFile(fileName, d, 0666)
    if err != nil {
        fmt.Println(&#34;write fail&#34;)
    }
    fmt.Println(&#34;write success&#34;)
}
```

- 写文件方式三：使用 File(Write,WriteString) 写入文件



```go
func Write2()  {

    fileName := &#34;file/test3&#34;
    strTest := &#34;测试测试&#34;
    var d1 = []byte(strTest)

    f, err3 := os.Create(fileName) //创建文件
    if err3 != nil{
        fmt.Println(&#34;create file fail&#34;)
    }
    defer f.Close()
    n2, err3 := f.Write(d1) //写入文件(字节数组)

    fmt.Printf(&#34;写入 %d 个字节n&#34;, n2)
    n3, err3 := f.WriteString(&#34;writesn&#34;) //写入文件(字节数组)
    fmt.Printf(&#34;写入 %d 个字节n&#34;, n3)
    f.Sync()
}
```

- 写文件方式四：使用 bufio.NewWriter 写入文件



```go
func Write3()  {
    fileName := &#34;file/test3&#34;
    f, err3 := os.Create(fileName) //创建文件
    if err3 != nil{
        fmt.Println(&#34;create file fail&#34;)
    }
    w := bufio.NewWriter(f) //创建新的 Writer 对象
    n4, err3 := w.WriteString(&#34;bufferedn&#34;)
    fmt.Printf(&#34;写入 %d 个字节n&#34;, n4)
    w.Flush()
    f.Close()
}
```

- 检查文件是否存在：



```go
func CheckFileExist(fileName string) bool {
    _, err := os.Stat(fileName)
    if os.IsNotExist(err) {
        return false
    }
    return true
}
```


---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175110/  

