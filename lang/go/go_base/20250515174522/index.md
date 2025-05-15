# 2-4. 数据类型相关操作

# 一. 数据互相转换

- int 转string `string := strconv.Itoa(int)`

- string到int `int,err := strconv.Atoi(string)`

- string到int64 `int64, err := strconv.ParseInt(string, 10, 64)`

- int64到string `string := strconv.FormatInt(int64,10)`

- map 到json字符串

  ```go
  m := map[string]string{}
  mjson,_ :=json.Marshal(m)
  mString :=string(mjson)
  ```

- json到struct 

  - 如果是单层可以直接用interface转为map

    ```go
    jsonStr = `{}`
    var jsonStruct interface{}
    JSON.Unmarshal([]byte(jsonStr), &amp;jsonStruct)
    ```

    

  - 有嵌套，把需要解析的字段一一列出，这里注意要首字母大写，否则无法解析。

    ```go
    type MyStruct struct{
      Name string `json:&#34;name&#34;`
    }
    jsonStr = `{name: 12}`
    var jsonStruct MyStruct{}
    JSON.Unmarshal([]byte(jsonStr), &amp;jsonStruct)
    ```

- 纯字符串拼接

  ```go
  a := &#34;hello&#34;
  b := &#34;world&#34;
  // 使用操作符
  result := a &#43; b
  // 使用Sprintf
  result := fmt.Sprintf(&#34;s%s%&#34;, a, b)
  // 使用join
  result := strings.Join([]string{a, b}, &#34;&#34;)
  // 使用buffer
  var buffer bytes.Buffer
  buffer.WriteString(a)
  buffer.WriteString(b)
  result := buffer.String()
  
  // 以上四种方式性能最好是buffer，实际开发中自行调节即可
  /*
  BenchmarkAddStringWithJoin-8       	20785686	        51.5 ns/op
  BenchmarkAddStringWithBuffer-8     	1000000000	        0.000123 ns/op
  BenchmarkAddStringWithSprintf-8    	 5727165	        196 ns/op
  BenchmarkAddStringWithOperator-8   	39417948	        29.1 ns/op
  
  
  */
  ```

  ```golang
  package main
   
  import (
      &#34;fmt&#34;
      &#34;strconv&#34;
  )
   
  func IntToString() {
      //todo :int to string
      v := 456
      vS := strconv.Itoa(v)
      fmt.Println(vS) //方法1，简便版
   
      //todo :int64 to string
      var vI64 int64 = 789
      vInt64S := strconv.FormatInt(vI64, 10) //方法2，int64转string，可指定几进制
      fmt.Println(vInt64S)
   
      //todo :uint64 to string
      var vUI64 uint64 = 91011
      vUI64S := strconv.FormatUint(vUI64, 10) //方法3， uint64转string，可指定几进制
      fmt.Println(vUI64S)
  }
   
  func StringToInt() {
      //todo :string to int/int64
      s := &#34;123&#34;
      vInt, _ := strconv.Atoi(s) //方法1，便捷版
      fmt.Println(vInt)
   
      vInt64, _ := strconv.ParseInt(s, 10, 64) //方案2，有符号整型，可以指定几进制，整数长度
      fmt.Println(vInt64)
   
      vUInt64, _ := strconv.ParseUint(s, 10, 64) //方案3，无符号整型，可以指定几进制，整数长度
      fmt.Println(vUInt64)
  }
   
  func StringToFloat() {
      //todo :string to float
      f64, _ := strconv.ParseFloat(&#34;123.456&#34;, 64) //方法1，可以指定长度
      fmt.Println(f64)
  }
   
  func FloatToString() {
      //todo :float to string
      f64 := 1223.13252
      sF64 := strconv.FormatFloat(f64, &#39;f&#39;, 5, 64) //方法1，可以指定输出格式、精度、长度
      fmt.Println(sF64)
  }
   
  func StringToBool() {
      //todo :string to bool
       接受 1, t, T, TRUE, true, True, 0, f, F, FALSE, false, False 等字符串；
       其他形式的字符串会返回错误
      b, _ := strconv.ParseBool(&#34;1&#34;)
      fmt.Println(b)
  }
  func BoolToString() {
      //todo :bool to string
      sBool := strconv.FormatBool(true) //方法1
      fmt.Println(sBool)
  }
   
  func main() {
      StringToInt()
      IntToString()
      StringToFloat()
      FloatToString()
      BoolToString()
      StringToBool()
  }
  ```
  
  

# 二. fmt占位符

|占位符 | 说明 | 举例 | 输出|
|---|---|---|---|
|**%v** | 相应值的默认格式。 | Printf(&#34;%v&#34;, people) | {zhangsan}|
|**%&#43;v** | 打印结构体时，会添加字段名 | Printf(&#34;%&#43;v&#34;, people) |{Name:zhangsan}|
|**%#v** | 相应值的Go语法表示 | Printf(&#34;#v&#34;, people) |main.Human{Name:&#34;zhangsan&#34;}|
|%T | 相应值的类型的Go语法表示 | Printf(&#34;%T&#34;, people) | main.Human|
|%% | 字面上的百分号，并非值的占位符 | Printf(&#34;%%&#34;) | %|
|**%t** | true 或 false。 | Printf(&#34;%t&#34;, true) | true|
|%b | 二进制表示 | Printf(&#34;%b&#34;, 5) | 101|
|%c | 相应Unicode码点所表示的字符 | Printf(&#34;%c&#34;, 0x4E2D) | 中|
|**%d** | 十进制表示 | Printf(&#34;%d&#34;, 0x12) | 18|
|%o | 八进制表示 | Printf(&#34;%d&#34;, 10) | 12|
|**%q** | 单引号围绕的字符字面值，由Go语法安全地转义| Printf(&#34;%q&#34;, 0x4E2D) |&#39;中&#39;|
|%x | 十六进制表示，字母形式为小写 a-f | Printf(&#34;%x&#34;, 13) | d|
|%X | 十六进制表示，字母形式为大写 A-F | Printf(&#34;%x&#34;, 13) | D|
|%U | Unicode格式：U&#43;1234，等同于 &#34;U&#43;%04X&#34; | Printf(&#34;%U&#34;, 0x4E2D) | U&#43;4E2D|
|%b | 无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat 的 &#39;b&#39; 转换格式一致。| | |
|%e | 科学计数法，例如 -1234.456e&#43;78 | Printf(&#34;%e&#34;, 10.2) |1.020000e&#43;01|
|%E | 科学计数法，例如 -1234.456E&#43;78 | Printf(&#34;%e&#34;, 10.2) |1.020000E&#43;01|
|%f | 有小数点而无指数，例如 123.456| Printf(&#34;%f&#34;, 10.2) | 10.200000|
|%g | 根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出| Printf(&#34;%g&#34;, 10.20) | 10.2|
|%G | 根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出 |Printf(&#34;%G&#34;, 10.20&#43;2i)| (10.2&#43;2i)|
|**%s** | 输出字符串表示（string类型或[]byte) | Printf(&#34;%s&#34;, []byte(&#34;Go语言&#34;)) |Go语言|
|**%q** | 双引号围绕的字符串，由Go语法安全地转义| Printf(&#34;%q&#34;, &#34;Go语言&#34;) | &#34;Go语言&#34;|
|%x | 十六进制，小写字母，每字节两个字符 | Printf(&#34;%x&#34;, &#34;golang&#34;) | 676f6c616e67|
|%X | 十六进制，大写字母，每字节两个字符 | Printf(&#34;%X&#34;, &#34;golang&#34;) | 676F6C616E67|
|%p | 十六进制表示，前缀 0x | Printf(&#34;%p&#34;, &amp;people) | 0x4f57f0|

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515174522/  

