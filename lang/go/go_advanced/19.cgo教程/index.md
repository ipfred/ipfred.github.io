# 

&gt; 官方文档： https://pkg.go.dev/cmd/cgo
&gt;
&gt; 参考：https://zhuanlan.zhihu.com/p/349197066、https://juejin.cn/post/7047405294107754533、https://pkg.go.dev/cmd/cgo

## 初识 cgo

- Cgo是Go语言中的一个工具，它允许**在Go代码中直接调用C语言代码**，并**让C语言代码调用Go代码**

- demo

  ```go
  package main
  
  /*
  #include &lt;stdio.h&gt;
  #include &lt;stdlib.h&gt;
  
  void myprint(char* s) {
    printf(&#34;%s\n&#34;, s);
  }
  */
  import &#34;C&#34;
  import &#34;unsafe&#34;
  
  func main() {
  	cs := C.CString(&#34;Hello from stdio&#34;)
  	C.myprint(cs)
  	C.free(unsafe.Pointer(cs))
  }
  // 输出： Hello from stdio
  ```

- 灰常神奇



## 在go中使用cgo

### 1. 开启cgo

- 使用`go env`查看, 确保 `CGO_ENABLED=1`

- 导入伪包“C”

  ```go
  import &#34;C&#34;
  ```

### 2. 在go里编写c代码

- 如果导入“C”之前紧跟着注释，则在编译包的 C 部分时，该注释（称为前导码）将用作标头; 

- 如果使用 C 标头，那么 注释后面要紧跟着 `import &#34;C&#34;`

  ```go
  // #include &lt;stdio.h&gt;
  // #include &lt;errno.h&gt;
  import &#34;C&#34;
  ```

- 可以在Go文件中通过Cgo的注释块来编写C代码。使用`/* */`将C代码包裹起来，将C代码直接插入Go代码中； 然后可以从 Go 代码中引用这些，就好像它们是在包“C”中定义的一样。可以使用序言中声明的所有名称，即使它们以小写字母开头； 但是前导码中的静态变量不能从 Go 代码中引用; 

  ```go
  package main
  
  /*
  #cgo LDFLAGS: -L/usr/local/lib
  
  #include &lt;stdio.h&gt;
  #include &lt;stdlib.h&gt;
  #define REPEAT_LIMIT 3              // CGO会保留C代码块中的宏定义
  typedef struct{                     // 自定义结构体
      int repeat_time;
      char* str;
  }blob;
  int SayHello(blob* pblob) {  // 自定义函数
      for ( ;pblob-&gt;repeat_time &lt; REPEAT_LIMIT; pblob-&gt;repeat_time&#43;&#43;){
          puts(pblob-&gt;str);
      }
      return 0;
  }
  */
  import &#34;C&#34;
  import (
      &#34;fmt&#34;
      &#34;unsafe&#34;
  )
  
  func main() {
      cblob := C.blob{}                               // 在GO程序中创建的C对象，存储在Go的内存空间
      cblob.repeat_time = 0
      cblob.str = C.CString(&#34;Hello, World\n&#34;)         // C.CString 会在C的内存空间申请一个C语言字符串对象，再将Go字符串拷贝到C字符串
      ret := C.SayHello(&amp;cblob)                       // &amp;cblob 取C语言对象cblob的地址
      fmt.Println(&#34;ret&#34;, ret)
      fmt.Println(&#34;repeat_time&#34;, cblob.repeat_time)
      C.free(unsafe.Pointer(cblob.str))               // C.CString 申请的C空间内存不会自动释放，需要显示调用C中的free释放
  }
  ```

  



## **CGO 的 N 种用法**

### 1. Go 调用自定义 C 程序

- demo

  ```go
  package main
  
  /*
  #include &lt;stdio.h&gt;
  #include &lt;stdlib.h&gt;
  
  void myprint(char* s) {
    printf(&#34;%s\n&#34;, s);
  }
  */
  import &#34;C&#34;
  import &#34;unsafe&#34;
  
  func main() {
  	cs := C.CString(&#34;Hello from stdio&#34;)
  	C.myprint(cs)
  	C.free(unsafe.Pointer(cs))  // 由于 C 的内存空间不受 Go 的 GC 管理，因此需要显示的调用 C 语言的 free 来进行回收
  }
  // 输出： Hello from stdio
  ```

  

### 2.  **Go 调用 C 模块**

- hello.c 文件

  ```c
  #include &lt;stdio.h&gt;
  int SayHello() {
      puts(&#34;Hello World&#34;);
      return 0;
  }
  ```

  

- main.go 

  &gt; main 中只对 `SayHello` 函数进行了声明，然后再通过链接 C 程序库的方式加载函数的实现

  ```go
  package main
  
  /*
  #include &#34;hello.c&#34;
  int SayHello();
  */
  import &#34;C&#34;
  import (
  	&#34;fmt&#34;
  )
  
  func main() {
  	ret := C.SayHello()  // Hello World
  	fmt.Println(ret)   // 0
  }
  
  ```

  

### 3. Go 调用 C&#43;&#43;模块

&gt; 通过**链接 C&#43;&#43;程序库**的方式，来实现 Go 调用 C&#43;&#43;程序

- hello.h

  ```c
  int SayHello();
  ```

- hello.cpp

  ```c&#43;&#43;
  #include &lt;iostream&gt;
  
  extern &#34;C&#34; {
      #include &#34;hello.h&#34;
  }
  
  int SayHello() {
   std::cout&lt;&lt;&#34;Hello World&#34;;
      return 0;
  }
  ```

- main.go

  &gt; CGO 提供的这种面向 C 语言接口的编程方式，使得开发者可以使用是任何编程语言来对接口进行实现，只要最终满足 C 语言接口即可。

  ```go
  package main
  /*
  #include &#34;hello.h&#34; 
  */
  import &#34;C&#34;
  import (
      &#34;fmt&#34;
  )
  
  func main() {
      ret := C.SayHello()
      fmt.Println(ret)
  }
  ```

### 4. go 调用C语言动态库

- 项目目录结构如下

  ```go
  ├── include
  │     └── add.c
  │     └── add.h
  ├── lib
  │     └── libadd.so
  └── main.go
  ```

  

- add.h

  ```c
  #ifndef __ADD_H__
  #define __ADD_H__
   
  char* Add(char* src, int n);
   
  #endif
  
  ```

- add.c

  ```c
  
  #include &lt;string.h&gt;
  #include &lt;stdio.h&gt;
  #include &lt;stdlib.h&gt;
   
  char* Add(char* src, int n)
  {
      char str[20];
      sprintf(str, &#34;%d&#34;, n);
      char *result = malloc(strlen(src)&#43;strlen(str)&#43;1);
      strcpy(result, src);
      strcat(result, str);
      return result;
  }
  
  ```

- linux 下编译

  &gt; 会在当前目录下生成 `libadd.so` 文件, 在 Linux 下可用 `nm -D libadd.so` 查看其中的方法

  ```
  gcc -fPIC -shared -o lib/libadd.so include/add.c
  ```

- main.go

  ```go
  package main
  
  /*
  // 头文件的位置，相对于源文件是当前目录，所以是 .，头文件在多个目录时写多个  #cgo CFLAGS: ...
  #cgo CFLAGS: -I./include
  // 从哪里加载动态库，位置与文件名，-ladd 加载 libadd.so 文件
  #cgo LDFLAGS: -L./lib -ladd -Wl,-rpath,lib
  #include &#34;add.h&#34;
  */
  import &#34;C&#34;
  import &#34;fmt&#34;
   
  func main() {
    val := C.Add(C.CString(&#34;go&#34;), 2023)
    fmt.Println(&#34;run c: &#34;, C.GoString(val))
  }
  
  ```

- 注意：

  - 如果把`#cgo LDFLAGS: -L./lib -ladd -Wl,-rpath,lib` 改为 `cgo LDFLAGS: -L./lib -ladd`编译不会报错，执行时会出错

    ```go
    error while loading shared libraries: libadd.so: cannot open shared object file: No such file or directory
    ```

  - 设置了环境变量 LD_LIBRARY_PATH=/home/.../lib 也能让它跑起来

    ```sh
    LD_LIBRARY_PATH=lib/ ./demo
    ```

    

### 5. C 调用 Go 模块

- C 调用 Go 相对于 Go 调 C 来说要复杂多，可以分为两种情况
  1. 一是原生 Go 进程调用 C，C 中再反调 Go 程序。
  2. 另一种是原生 C 进程直接调用 Go。

#### 5.1 go 实现c的函数

&gt; Go 程序先调用 C 的 SayHello 接口，由于 SayHello 接口链接在 Go 的实现上，又调到 Go。
&gt;
&gt; 看起来调起方和实现方都是 Go，但实际执行顺序是 Go 的 main 函数，调到 CGO 生成的 C 桥接函数，最后 C 桥接函数再调到 Go 的 SayHello

- demo/hello.h

  ```c
  void SayHello(char* s);
  ```

- demo/hello.go

  &gt; CGO 的//export SayHello 指令将 Go 语言实现的 SayHello 函数导出为 C 语言函数。这样再 Go 中调用 C.SayHello 时，最终调用的是 hello.go 中定义的 Go 函数 SayHello

  ```go
  package main
  
  // #include &lt;hello.h&gt;
  import &#34;C&#34;
  import &#34;fmt&#34;
  
  //export SayHello   
  func SayHello(str *C.char) {
  	fmt.Println(C.GoString(str))
  }
  
  ```

- demo/main.go

  &gt; go run ..\demo\

  ```go
  package main
  
  // #include &#34;hello.h&#34;
  import &#34;C&#34;
  
  func main() {
  	C.SayHello(C.CString(&#34;Hello World&#34;))  // Hello World
  }
  
  ```

  

#### 5.2 原生 C 调用 Go

- hello.go

  ```go
  package main
  
  import &#34;C&#34;
  
  //export hello
  func hello(value string)*C.char {   // 如果函数有返回值，则要将返回值转换为C语言对应的类型;如果 Go 函数有多个返回值，会生成一个 C 结构体进行返回，结构体定义参考生成的.h 文件
      return C.CString(&#34;hello&#34; &#43; value)
  }
  func main(){
      // 此处一定要有main函数，有main函数才能让cgo编译器去把包编译成C的库
  }
  ```

- 生成c-shared文件 `go build -buildmode=c-shared -o hello.so hello.go`

- hello.c

  ```c
  #include &lt;stdio.h&gt;
  #include &lt;string.h&gt;
  #include &#34;hello.h&#34;                       //此处为上一步生成的.h文件
  
  int main(){
      char c1[] = &#34;did&#34;;
      GoString s1 = {c1,strlen(c1)};       //构建Go语言的字符串类型
      char *c = hello(s1);
      printf(&#34;r:%s&#34;,c);
      return 0;
  }
  ```

- 编译`gcc -o c_go main.c hello.so`

## CFLAGS 与 LDFLAGS

- 参数含义

  - **CFLAGS ** : 头文件的位置，相对于源文件是当前目录;头文件在多个目录时写多个  #cgo CFLAGS: ...
  - **LDFLAGS** ：从哪里加载动态库，位置与文件名，e.g.: `-ladd` 加载 `libadd.so` 文件

  ```go
  /*
  // 头文件的位置，相对于源文件是当前目录，所以是 .，头文件在多个目录时写多个  #cgo CFLAGS: ...
  #cgo CFLAGS: -I./include
  // 从哪里加载动态库，位置与文件名，-ladd 加载 libadd.so 文件
  #cgo LDFLAGS: -L./lib -ladd -Wl,-rpath,lib
  #include &#34;add.h&#34;
  */
  ```

  &gt; 在指定目录找不到对应的文件或者库时会报错！！！

- 软件包中的所有 cgo `CPPFLAG` 和 `CFLAGS` 指令都连接起来并用于编译该软件包中的 C 文件。包中的所有 `CPPFLAGS` 和 `CXXFLAGS` 指令都连接起来，用于编译该包中的C&#43;&#43;文件

- 解析 cgo 指令时，任何出现的字符串 ${SRCDIR} 都将替换为包含源文件的目录的绝对路径。这允许将预编译的静态库包含在包目录中并正确链接。例如，如果 package foo 位于目录 /go/src/foo 中：

  ```go
  // #cgo LDFLAGS: -L${SRCDIR}/libs -lfoo
  将扩展到=&gt;
  // #cgo LDFLAGS: -L/go/src/foo/libs -lfoo
  ```

  

## CGO 与 Go类型转换

- 标准 Cgo 类型 `C.char`、`C.schar（有符号 char）`、`C.uchar （无符号字符）`、`C.short`、`C.ushort （无符号短）`、`C.int`、`C.uint（无符号整数）`、`C.long`、`C.ulong （无符号长）`、`C.longlong（长长）`、`C.ulonglong（无符号长长）`、`C.float`、`C.double`、`C.complexfloat（复数浮点数）`和` C.complexdouble（复数双精度）`

- 对照关系

  | C类型                  | Cgo类型                  | go类型         | 字节数（byte） | 数值范围                                 |
  | ---------------------- | ------------------------ | -------------- | -------------- | ---------------------------------------- |
  | char                   | C.char                   | byte           | 1              | -128~127                                 |
  | signed char            | C.schar                  | int8           | 1              | -128~127                                 |
  | unsigned char          | C.uchar                  | uint8          | 1              | 0~255                                    |
  | short int              | C.short                  | int16          | 2              | -32768~32767                             |
  | short unsigned int     | C.ushort                 | uint16         | 2              | 0~65535                                  |
  | int                    | C.int                    | int            | 4              | -2147483648~2147483647                   |
  | unsigned int           | C.uint                   | uint32         | 4              | 0~4294967295                             |
  | long int               | C.long                   | int32 or int64 | 4              | -2147483648~2147483647                   |
  | long unsigned int      | C.ulong uint32 or uint64 | 4              | 0~4294967295   |                                          |
  | long long int          | C.longlong               | int64          | 8              | -9223372036854776001~9223372036854775999 |
  | long long unsigned int | C.ulonglong              | uint64         | 8              | 0~18446744073709552000                   |
  | float                  | C.float                  | float32        | 4              | -3.4E-38~3.4E&#43;38                         |
  | double                 | C.double                 | float64        | 8              | 1.7E-308~1.7E&#43;308                        |
  | wchar_t                | C.wchar_t                | wchar_t        | 2              | 0~65535                                  |
  | void *                 | unsafe.Pointer           |                |                |                                          |

- Go 语言的 int 和 uint 在 32 位和 64 位系统下分别是 4 个字节和 8 个字节大小。它在 C 语言中的导出类型 GoInt 和 GoUint 在不同位数系统下内存大小也不同。如下是 64 位系统中，Go 数值类型在 C 语言的导出列表

  ```go
  // _cgo_export.h
  typedef signed char GoInt8;
  typedef unsigned char GoUint8;
  typedef short GoInt16;
  typedef unsigned short GoUint16;
  typedef int GoInt32;
  typedef unsigned int GoUint32;
  typedef long long GoInt64;
  typedef unsigned long long GoUint64;
  typedef GoInt64 GoInt;
  typedef GoUint64 GoUint;
  typedef __SIZE_TYPE__ GoUintptr;
  typedef float GoFloat32;
  typedef double GoFloat64;
  typedef float _Complex GoComplex64;
  typedef double _Complex GoComplex128;
  ```

- 需要注意的是**在 C 语言符号名前加上 \*Ctype\*， 便是其在 Go 中的导出名，因此在启用 CGO 特性后，Go 语言中禁止出现以\*Ctype\* 开头的自定义符号名，类似的还有\*Cfunc\*等**

### 1. 切片

- Go 中切片的使用方法类似 C 中的数组，但是内存结构并不一样

- C 中的数组实际上指的是一段连续的内存，而 Go 的切片在存储数据的连续内存基础上，还有一个头结构体，其内存结构如下

  ![img](https://pic4.zhimg.com/80/v2-6d1ec793316e5ad735835197408c774b_720w.webp)

- 因此 Go 的切片不能直接传递给 C 使用，而是需要取切片的内部缓冲区的首地址(即首个元素的地址)来传递给 C 使用。使用这种方式把 Go 的内存空间暴露给 C 使用，可以大大减少 Go 和 C 之间参数传递时内存拷贝的消耗。

- demo

  ```go
  package main
  
  /*
  int SayHello(char* buff, int len) {
      char hello[] = &#34;Hello Cgo!&#34;;
      int movnum = len &lt; sizeof(hello) ? len:sizeof(hello);
      memcpy(buff, hello, movnum);                        // go字符串没有&#39;\0&#39;，所以直接内存拷贝
      return movnum;
  }
  
  */
  import &#34;C&#34;
  import (
      &#34;fmt&#34;
      &#34;unsafe&#34;
  )
  
  func main() {
      buff := make([]byte, 8)
      C.SayHello((*C.char)(unsafe.Pointer(&amp;buff[0])), C.int(len(buff)))
      a := string(buff)
      fmt.Println(a)
  }
  ```

  

### 2. 字符串

- Go 的字符串与 C 的字符串在底层的内存模型不一样：

  ![img](https://pic3.zhimg.com/80/v2-6a41aeb589548c13551feb8790094776_720w.webp)

- Go 的字符串并没有以&#39;\0&#39; 结尾，因此使用类似切片的方式，直接将 Go 字符串的首元素地址传递给 C 是不可行的

- cgo 给出的解决方案是标准库函数 C.CString()，它会在 C 内存空间内申请足够的空间，并将 Go 字符串拷贝到 C 空间中。因此 C.CString 申请的内存在 C 空间中，因此需要显式的调用 C.free 来释放空间

  - C.CString()的底层实现

    ```go
    func _Cfunc_CString(s string) *_Ctype_char {        // 从Go string 到 C char* 类型转换
     p := _cgo_cmalloc(uint64(len(s)&#43;1))
     pp := (*[1&lt;&lt;30]byte)(p)
     copy(pp[:], s)
     pp[len(s)] = 0
     return (*_Ctype_char)(p)
    }
    
    //go:cgo_unsafe_args
    func _cgo_cmalloc(p0 uint64) (r1 unsafe.Pointer) {
     _cgo_runtime_cgocall(_cgo_bb7421b6328a_Cfunc__Cmalloc, uintptr(unsafe.Pointer(&amp;p0)))
     if r1 == nil {
      runtime_throw(&#34;runtime: C malloc failed&#34;)
     }
     return
    }
    ```

- 更高效的字符串传递方法

  - C.CString 简单安全，但是它涉及了一次从 Go 到 C 空间的内存拷贝，对于长字符串而言这会是难以忽视的开销。

  - Go 官方文档中声称 string 类型是”不可改变的“，但是在实操中可以发现，除了常量字符串会在编译期被分配到只读段，其他的动态生成的字符串实际上都是在堆上。

  - 因此如果能够获得 string 的内存缓存区地址，那么就可以使用类似切片传递的方式将字符串指针和长度直接传递给 C 使用。

  - 查阅源码，可知 String 实际上是由缓冲区首地址 和 长度构成的。这样就可以通过一些方式拿到缓存区地址。

    ```text
    type stringStruct struct {
     str unsafe.Pointer  //str首地址
     len int             //str长度
    }
    ```

  - test11.go 将 fmt 动态生成的 string 转为自定义类型 MyString 便可以获得缓冲区首地址，将地址传入 C 函数，这样就可以在 C 空间直接操作 Go-String 的内存空间了，这样可以免去内存拷贝的消耗。

    ```text
    // test11.go
    package main
    
    /*
    #include &lt;string.h&gt;
    int SayHello(char* buff, int len) {
        char hello[] = &#34;Hello Cgo!&#34;;
        int movnum = len &lt; sizeof(hello) ? len:sizeof(hello);
        memcpy(buff, hello, movnum);
        return movnum;
    }
    */
    import &#34;C&#34;
    import (
        &#34;fmt&#34;
        &#34;unsafe&#34;
    )
    
    type MyString struct {
     Str *C.char
     Len int
    }
    func main() {
        s := fmt.Sprintf(&#34;             &#34;)
        C.SayHello((*MyString)(unsafe.Pointer(&amp;s)).Str, C.int((*MyString)(unsafe.Pointer(&amp;s)).Len))
        fmt.Print(s)
    }
    ```

  - **这种方法背离了 Go 语言的设计理念，如非必要，不要把这种代码带入你的工程，这里只是作为一种“黑科技”进行分享。**

  

### 3. 结构体，联合，枚举

- cgo 中结构体，联合，枚举的使用方式类似，可以通过 C.struct_XXX 来访问 C 语言中 struct XXX 类型。union,enum 也类似

#### 3.1 **结构体**

- 如果结构体的成员名字中碰巧是 Go 语言的关键字，可以通过在成员名开头添加下划线来访问

- 如果有 2 个成员：一个是以 Go 语言关键字命名，另一个刚好是以下划线和 Go 语言关键字命名，那么以 Go 语言关键字命名的成员将无法访问（被屏蔽）

- C 语言结构体中位字段对应的成员无法在 Go 语言中访问，如果需要操作位字段成员，需要通过在 C 语言中定义辅助函数来完成。对应零长数组的成员(C 中经典的变长数组)，无法在 Go 语言中直接访问数组的元素，但同样可以通过在 C 中定义辅助函数来访问。

- 结构体的内存布局按照 C 语言的通用对齐规则，在 32 位 Go 语言环境 C 语言结构体也按照 32 位对齐规则，在 64 位 Go 语言环境按照 64 位的对齐规则。**对于指定了特殊对齐规则的结构体，无法在 CGO 中访问。**

- demo

  ```go
  package main
  /*
  struct Test {
      int a;
      float b;
      double type;
      int size:10;
      int arr1[10];
      int arr2[];
  };
  int Test_arr2_helper(struct Test * tm ,int pos){
      return tm-&gt;arr2[pos];
  }
  #pragma  pack(1)
  struct Test2 {
      float a;
      char b;
      int c;
  };
  */
  import &#34;C&#34;
  import &#34;fmt&#34;
  func main() {
      test := C.struct_Test{}
      fmt.Println(test.a)
      fmt.Println(test.b)
      fmt.Println(test._type)
      //fmt.Println(test.size)        // 位数据
      fmt.Println(test.arr1[0])
      //fmt.Println(test.arr)         // 零长数组无法直接访问
      //Test_arr2_helper(&amp;test, 1)
  
      test2 := C.struct_Test2{}
      fmt.Println(test2.c)
      //fmt.Println(test2.c)          // 由于内存对齐，该结构体部分字段Go无法访问
  }
  ```

  

#### 3.2 联合

- Go 语言中并不支持 C 语言联合类型，它们会被转为对应大小的字节数组。

- 如果需要操作 C 语言的联合类型变量，一般有三种方法：第一种是在 C 语言中定义辅助函数；第二种是通过 Go 语言的&#34;encoding/binary&#34;手工解码成员(需要注意大端小端问题)；第三种是使用`unsafe`包强制转型为对应类型(这是性能最好的方式)。

- demo

  ```go
  package main
  /*
  #include &lt;stdint.h&gt;
  union SayHello {
   int Say;
   float Hello;
  };
  union SayHello init_sayhello(){
      union SayHello us;
      us.Say = 100;
      return us;
  }
  int SayHello_Say_helper(union SayHello * us){
      return us-&gt;Say;
  }
  */
  import &#34;C&#34;
  import (
      &#34;fmt&#34;
      &#34;unsafe&#34;
      &#34;encoding/binary&#34;
  )
  
  func main() {
      SayHello := C.init_sayhello()
      fmt.Println(&#34;C-helper &#34;,C.SayHello_Say_helper(&amp;SayHello))           // 通过C辅助函数
      buff := C.GoBytes(unsafe.Pointer(&amp;SayHello), 4)
      Say2 := binary.LittleEndian.Uint32(buff)
      fmt.Println(&#34;binary &#34;,Say2)                 // 从内存直接解码一个int32
      fmt.Println(&#34;unsafe modify &#34;, *(*C.int)(unsafe.Pointer(&amp;SayHello)))     // 强制类型转换
  }
  ```

  

#### 3.3 枚举

- 对于枚举类型，可以通过`C.enum_xxx`来访问 C 语言中定义的`enum xxx`结构体类型, 使用方式和 C 相同

### 4. 指针

- 如果一个指针类型是用 type 命令在另一个指针类型基础之上构建的，换言之两个指针底层是相同完全结构的指针，那么也可以通过直接强制转换语法进行指针间的转换。在 Go 语言中两个指针的类型完全一致则不需要转换可以直接通用。

- 但是 C 语言中，不同类型的指针是可以显式或隐式转换。cgo 经常要面对的是 2 个完全不同类型的指针间的转换，实现这一转换的关键就是 unsafe.Pointer,类似于 C 语言中的 Void*类型指针

  ![img](https://pic3.zhimg.com/80/v2-36d6e0c7b13fb0b46177d94535d3f0d6_720w.webp)

- 使用这种方式就可以实现不同类型间的转换，如下是从 Go - int32 到 *C.char 的转换。


---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/19.cgo%E6%95%99%E7%A8%8B/  

