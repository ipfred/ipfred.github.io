# 20.go调用lib和so动态库

## go 调用dll

### 1. sysCall.LoadDll(推荐使用)

- 系统调用是程序向操作系统内核请求服务的过程，通常包含硬件相关的服务(例如访问硬盘),创建新进程等。系统调用提供了一个进程和操作系统之间的接口

- fmt中的syscall

  ```GO
  func Println(a ...interface{}) (n int, err error) {
      return Fprintln(os.Stdout, a...)
  }
  
  Stdout = NewFile(uintptr(syscall.Stdout), &#34;/dev/stdout&#34;)
  ```

- 调用dll 示例

  ```go
  dll, err := syscall.LoadDLL(&#34;scan.dll&#34;)
  //根据名称从dll中查找proc
  MemoryStream_Get = dll.FindProc(&#34;AllocateMemory&#34;)
  MemoryStream_Get.Call()
  ```

- 此方式可以 也可以调用go 代码打包的dll

### 2. Cgo调用

- 项目目录结构如下

  ```go
  ├── include
  │     └── add.c
  │     └── add.h
  ├── lib
  │     └── libadd.dll
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

- main.go

  ```go
  func GetFinalStrategyString(request string) {
  
  	dll, err := syscall.LoadDLL(&#34;./middleware_c.dll&#34;)
  	if err != nil {
  		log.Fatal(&#34;Error loading DLL:&#34;, err)
  	}
  	defer dll.Release()
  
  	getFinalStrategyString, err := dll.FindProc(&#34;GetFinalStrategyString&#34;)
  	if err != nil {
  		log.Fatal(&#34;Error finding GetFinalStrategyString:&#34;, err)
  	}
  
  	freeFinalStrategyString, err := dll.FindProc(&#34;FreeFinalStrategyString&#34;)
  	if err != nil {
  		log.Fatal(&#34;Error finding FreeFinalStrategyString:&#34;, err)
  	}
  
  	// cRequest, err := syscall.UTF16PtrFromString(request)
  	// if err != nil {
  	// 	log.Fatal(&#34;Error converting request:&#34;, err)
  	// }
  
  	var cResponse *uint16
  	var responseLen uint32
  
  	ret, _, _ := getFinalStrategyString.Call(
  		uintptr(unsafe.Pointer(request)),
  		uintptr(len(request)),
  		uintptr(unsafe.Pointer(&amp;cResponse)),
  		uintptr(unsafe.Pointer(&amp;responseLen)),
  	)
  
  	if ret != 0 {
  		log.Fatal(&#34;ret Error:&#34;, ret)
  	}
  	fmt.Println(&#34;responseLen:&#34;, responseLen)
  	fmt.Println(&#34;cResponse:&#34;, cResponse)
  
  	cResponseBytes := (*[1 &lt;&lt; 20]byte)(unsafe.Pointer(cResponse))[:responseLen]
  	fmt.Println(&#34;cResponseBytes:&#34;, cResponseBytes)
  }
  
  ```

  

### 3. 动态调用 dll （推荐使用）

- 动态调用步骤
  1. 通过文件路径加载c/c&#43;&#43; 动态库中的 handle
  2. 在go代码中定义和c/c&#43;&#43; 动态库中对应的go func 
  3. 使用库purego（底层是dlopen）或者sys/windows 将handle中对应的方法符号（Symbol）映射到 go func 地址上
  4. 逻辑调用

- main.go

  ```go
  import &#34;C&#34;
  import (
  	&#34;context&#34;
  	&#34;fmt&#34;
  	&#34;github.com/ebitengine/purego&#34;
  	&#34;sync&#34;
  )
  
  
  var (
  	getFinalStrategyString func(*C.char, C.uint, **C.char, *C.uint) C.int  //2. 定义和c/c&#43;&#43; 动态库中对应的go func
  	freeFinalStrategyString func(*C.char)  // 2. 定义和c/c&#43;&#43; 动态库中对应的go func
  	mcRunModeOnce sync.Once
  	mcRunMode     int
  )
  
  func main(){
      InitMC(libFilePath)
  }
  
  func InitMC(libFilePath string) (err error) {
  	defer func() {
  		if _err := recover(); _err != nil {
  			logger.Errorf(&#34;RegisterLibFunc panic error: %v&#34;, err)
  			err = fmt.Errorf(&#34;RegisterLibFunc panic error: %v&#34;, err.Error())
  		}
  	}()
  	...
  	mcHandle, err := LoadHandle(libFilePath)  // 1. 通过路径加载c/c&#43;&#43; 动态库handle
  	if err != nil {
  		logger.Errorf(&#34;LoadHandle error:%v&#34;, err.Error())
  		return err
  	}
      // 找不到函数会panic
  	purego.RegisterLibFunc(&amp;getFinalStrategyString, mcHandle, &#34;GetFinalStrategyString&#34;) // 3. 使用库purego将handle中对应的方法映射到go func
  	purego.RegisterLibFunc(&amp;freeFinalStrategyString, mcHandle, &#34;FreeFinalStrategyString&#34;)  // 3. 使用库purego将handle中对应的方法映射到go func
  	return nil
  }
  
  
  ```

  

- load_linux.go

  ```go
  //go:build !windows
  // &#43;build !windows
  
  package main
  
  import &#34;github.com/ebitengine/purego&#34;
  
  const MCLibPath = &#34;c.so&#34;
  
  func LoadHandle(libPath string) (uintptr, error) {
  	return purego.Dlopen(libPath, purego.RTLD_NOW|purego.RTLD_GLOBAL)
  }
  
  ```

- load_windows.go

  ```go
  //go:build windows
  // &#43;build windows
  
  package main
  
  import &#34;golang.org/x/sys/windows&#34;
  
  const MCLibPath = &#34;&#34;
  
  func LoadHandle(libPath string) (uintptr, error) {
  	handle, err := windows.LoadLibrary(libPath)
  	return uintptr(handle), err
  }
  
  ```

- 业务调用

  &gt; c&#43;&#43; 头文件
  &gt;
  &gt; \#ifdef __cplusplus
  &gt; extern &#34;C&#34; {
  &gt; \#endif
  &gt;
  &gt; MIDDLEWARE_C_EXPORT int GetFinalStrategyString(
  &gt;     /*::browserconsoleapiv3::middle_ware::GetUserStrategiesRequest request*/
  &gt;     const char* request,
  &gt;     unsigned request_len,
  &gt;     /*::browserconsoleapiv3::api::user::FinalGroupStrategy response*/
  &gt;     char** response,
  &gt;     unsigned* response_len);
  &gt;
  &gt; MIDDLEWARE_C_EXPORT void FreeFinalStrategyString(const char*);
  &gt;
  &gt; \#ifdef __cplusplus
  &gt; }
  &gt; \#endif

  ```go
  package middleware
  
  /*
  #include &lt;stdlib.h&gt;  // 要引入
  */
  import &#34;C&#34;
  import (
  	&#34;fmt&#34;
  	&#34;unsafe&#34;
  )
  
  type MiddlewareC struct {
  	response *C.char
  }
  
  func (mc *MiddlewareC) GetFinalStrategyString(req string) (resByte []byte, err error) {
  	cRequest := C.CString(req)
  	defer C.free(unsafe.Pointer(cRequest))
  	var (
  		reqLen      C.uint
  		cResponse   *C.char
  		responseLen C.uint
  		resCodeC    C.int
  	)
  	reqLen = C.uint(len(req))
  	resCodeC = getFinalStrategyString(cRequest, reqLen, &amp;cResponse, &amp;responseLen)  //4. 真正的业务调用
  	resCode := int(resCodeC)
  	if resCode != 0 {
  		errMsg, ok := MCError[resCode]
  		if !ok {
  			err = fmt.Errorf(
  				&#34;unknown error code, code:%v, cResponse: %v, cResponse len: %v&#34;,
  				resCode, cResponse, responseLen,
  			)
  		} else {
  			err = fmt.Errorf(
  				&#34;middleware error msg: %v, error code: %v, cResponse: %v, cResponse len: %v&#34;,
  				errMsg, resCode, cResponse, responseLen,
  			)
  		}
  		return resByte, err
  	}
  	resByte = C.GoBytes(unsafe.Pointer(cResponse), C.int(responseLen))
  	mc.response = cResponse
  	return resByte, nil
  }
  
  func (mc *MiddlewareC) MustFreeFinalStrategyString() {
  	freeFinalStrategyString(mc.response)
  }
  
  ```

  

## go 调用so

### 1. Cgo调用

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

    

### 2. 动态调用 so（推荐使用）

- 动态调用步骤
  1. 通过路径加载c/c&#43;&#43; 动态库 handle
  2. 定义和c/c&#43;&#43; 动态库中对应的go func
  3. 使用库purego 将handle中对应的方法映射到go func
  4. 逻辑调用

- main.go

  ```go
  import &#34;C&#34;
  import (
  	&#34;context&#34;
  	&#34;fmt&#34;
  	&#34;github.com/ebitengine/purego&#34;
  	&#34;sync&#34;
  )
  
  
  var (
  	getFinalStrategyString func(*C.char, C.uint, **C.char, *C.uint) C.int  //2. 定义和c/c&#43;&#43; 动态库中对应的go func
  	freeFinalStrategyString func(*C.char)  // 2. 定义和c/c&#43;&#43; 动态库中对应的go func
  	mcRunModeOnce sync.Once
  	mcRunMode     int
  )
  
  func main(){
      InitMC(libFilePath)
  }
  
  func InitMC(libFilePath string) (err error) {
  	defer func() {
  		if _err := recover(); _err != nil {
  			logger.Errorf(&#34;RegisterLibFunc panic error: %v&#34;, err)
  			err = fmt.Errorf(&#34;RegisterLibFunc panic error: %v&#34;, err.Error())
  		}
  	}()
  	...
  	mcHandle, err := LoadHandle(libFilePath)  // 1. 通过路径加载c/c&#43;&#43; 动态库handle
  	if err != nil {
  		logger.Errorf(&#34;LoadHandle error:%v&#34;, err.Error())
  		return err
  	}
      // 找不到函数会panic
  	purego.RegisterLibFunc(&amp;getFinalStrategyString, mcHandle, &#34;GetFinalStrategyString&#34;) // 3. 使用库purego将handle中对应的方法映射到go func
  	purego.RegisterLibFunc(&amp;freeFinalStrategyString, mcHandle, &#34;FreeFinalStrategyString&#34;)  // 3. 使用库purego将handle中对应的方法映射到go func
  	return nil
  }
  
  
  ```

  

- load_linux.go

  ```go
  //go:build !windows
  // &#43;build !windows
  
  package main
  
  import &#34;github.com/ebitengine/purego&#34;
  
  const MCLibPath = &#34;c.so&#34;
  
  func LoadHandle(libPath string) (uintptr, error) {
  	return purego.Dlopen(libPath, purego.RTLD_NOW|purego.RTLD_GLOBAL)
  }
  
  ```

- load_windows.go

  ```go
  //go:build windows
  // &#43;build windows
  
  package main
  
  import &#34;golang.org/x/sys/windows&#34;
  
  const MCLibPath = &#34;&#34;
  
  func LoadHandle(libPath string) (uintptr, error) {
  	handle, err := windows.LoadLibrary(libPath)
  	return uintptr(handle), err
  }
  
  ```

- 业务调用

  &gt; c&#43;&#43; 头文件
  &gt;
  &gt; \#ifdef __cplusplus
  &gt; extern &#34;C&#34; {
  &gt; \#endif
  &gt;
  &gt; MIDDLEWARE_C_EXPORT int GetFinalStrategyString(
  &gt;     /*::browserconsoleapiv3::middle_ware::GetUserStrategiesRequest request*/
  &gt;     const char* request,
  &gt;     unsigned request_len,
  &gt;     /*::browserconsoleapiv3::api::user::FinalGroupStrategy response*/
  &gt;     char** response,
  &gt;     unsigned* response_len);
  &gt;
  &gt; MIDDLEWARE_C_EXPORT void FreeFinalStrategyString(const char*);
  &gt;
  &gt; \#ifdef __cplusplus
  &gt; }
  &gt; \#endif

  ```go
  package middleware
  
  /*
  #include &lt;stdlib.h&gt;  // 要引入
  */
  import &#34;C&#34;
  import (
  	&#34;fmt&#34;
  	&#34;unsafe&#34;
  )
  
  type MiddlewareC struct {
  	response *C.char
  }
  
  func (mc *MiddlewareC) GetFinalStrategyString(req string) (resByte []byte, err error) {
  	cRequest := C.CString(req)
  	defer C.free(unsafe.Pointer(cRequest))
  	var (
  		reqLen      C.uint
  		cResponse   *C.char
  		responseLen C.uint
  		resCodeC    C.int
  	)
  	reqLen = C.uint(len(req))
  	resCodeC = getFinalStrategyString(cRequest, reqLen, &amp;cResponse, &amp;responseLen)  //4. 真正的业务调用
  	resCode := int(resCodeC)
  	if resCode != 0 {
  		errMsg, ok := MCError[resCode]
  		if !ok {
  			err = fmt.Errorf(
  				&#34;unknown error code, code:%v, cResponse: %v, cResponse len: %v&#34;,
  				resCode, cResponse, responseLen,
  			)
  		} else {
  			err = fmt.Errorf(
  				&#34;middleware error msg: %v, error code: %v, cResponse: %v, cResponse len: %v&#34;,
  				errMsg, resCode, cResponse, responseLen,
  			)
  		}
  		return resByte, err
  	}
  	resByte = C.GoBytes(unsafe.Pointer(cResponse), C.int(responseLen))
  	mc.response = cResponse
  	return resByte, nil
  }
  
  func (mc *MiddlewareC) MustFreeFinalStrategyString() {
  	freeFinalStrategyString(mc.response)
  }
  
  ```

  

## 大坑！！！

- 动态库中的崩溃会直接导致主程序崩溃！！！！！！！！！！

- 崩溃测试

  - 创建崩溃程序

    ```go
    package main
    
    import &#34;C&#34;
    import &#34;fmt&#34;
    
    //export PrintTest
    func PrintTest() int {
    	defer func() {
    		if err := recover(); err != nil {
    			print(&#34;err\n&#34;)
    			print(err)
    		}
    	}()
    	print(&#34;hello world\n&#34;)
    	var a []int
    	a[0] = 1
    	print(&#34;hello world 2\n&#34;)
    	return 1
    }
    
    func main() {
    	fmt.Println(&#34;call cpp test&#34;)
    }
    
    ```

  - 打包dll文件`go build -o pt.dll -buildmode=c-shared main.go` 

  - 主程序调用

    ```go
    package main
    
    import &#34;C&#34;
    import (
    	&#34;github.com/ebitengine/purego&#34;
    	win &#34;golang.org/x/sys/windows&#34;
    	&#34;log&#34;
    )
    
    var PrintTest func() int
    
    func openLibrary(name string) (uintptr, error) {
    	handle, err := win.LoadLibrary(name)
    	return uintptr(handle), err
    }
    
    func main() {
        // recover无效了！
    	defer func() {
    		if err := recover(); err != nil {
    			log.Println(&#34;err&#34;)
    			log.Println(err)
    		}
    	}()
    
    	lib, err := openLibrary(&#34;./dll/pt.dll&#34;)
    	if err != nil {
    		log.Fatalln(err)
    	}
    	purego.RegisterLibFunc(&amp;PrintTest, lib, &#34;PrintTest&#34;)
    	res := PrintTest()
    	log.Println(res)
    }
    
    // ！！！ 直接panic，recover无效
    ```
    
    

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180311/  

