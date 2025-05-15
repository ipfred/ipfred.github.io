# 8. nethttp模块

- Go语言内置的`net/http`包十分的优秀，

- 压力测试数据

  ```shell
  # mac 配置 8核 16G内存
  # goland 多核模式
    16 threads and 200 connections
    Thread Stats   Avg      Stdev     Max   &#43;/- Stdev
      Latency     5.75ms   14.07ms 224.91ms   90.89%
      Req/Sec     9.46k     6.90k  100.80k    80.15%
    4403567 requests in 30.09s, 596.34MB read
    Socket errors: connect 0, read 56, write 0, timeout 0
  Requests/sec: 146360.93 # 每秒并发数高达 14.6w  是sanic 10进程的3倍
  Transfer/sec:  19.82MB
  ```

- HTTP客户端有两个非常重要的类型client和request

## 1. Client

- Client 结构体共有四个成员
  1. Transport 指定独立单次HTTP请求的机制
  2. CheckRedirect 指定处理重定向策略
  3. Jar 指定cookie管理器
  4. Timeout 指定文本类型的执行请求的时间限制

```go
import &#34;time&#34;

type Client struct {
	Transport RoundTripper
	CheckRedirect func(req *Request, via []*Request)error
	Jar CookieJar
	Timeout time.Duration

}
```

- Client类型主要充当浏览器角色; 它拥有一下方法:
  - Do   *get和post都是基于do方法进行的封装*
  - Head
  - Get
  - Post
  - PostForm

&lt;img src=&#34;https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211206214457663.png&#34; alt=&#34;image-20211206214457663&#34; style=&#34;zoom:50%;&#34; /&gt;

### 1.1 启动client

![image-20211206214745422](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20211206214745422.png)

```go
/*
 * @date: 2021/12/6
 * @desc: ...
 */

package main

import (
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net&#34;
	&#34;net/http&#34;
	&#34;time&#34;
)

func main() {
	// 创建连接池
	transport := &amp;http.Transport{
		DialContext: (&amp;net.Dialer{
			Timeout:   30 * time.Second, // 连接超时
			KeepAlive: 30 * time.Second, // 长连接保持时间
		}).DialContext,
		MaxIdleConns:          100,              // 最大空闲连接
		IdleConnTimeout:       90 * time.Second, //空闲超时时间
    TLSHandshakeTimeout:   10 * time.Second, //tls 握手超时时间 (https使用)
		ExpectContinueTimeout: 1 * time.Second,  //100-continue状态码超时时间
	}
	// 创建客户端
	client := http.Client{
		Timeout:   time.Second * 30, // 请求超时时间
		Transport: transport,
	}
	// 请求数据
	resp, err := client.Get(&#34;http://127.0.0.1:8080/bye&#34;)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	// 读取body内容
	bds, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(bds))

}

```



## 2. Request

- request是对请求体的封装吗任何形式的http请求都可以由request来构造, 构造完成之后使用Client发送请求;

  ```go
  type Request struct {
  	Method string		//请求方法
  	Url *url.URL 		//请求地址
  	Proto string 		//协议版本
  	Header Header 		//请求头
  	Body io.ReadCloser 	//请求体
  	Form url.Values 	//解析好的表单数据,包括URL字段的query参数和post或者put的表单数据
  	//...
  	
  }
  ```

  

## 3. Get和Post的区别

1. get是从服务器上获取数据, post是向服务器推送数据
2. get和post传递参数的方式不同, get将参数放在url后面,post是将表单数据放在请求体中
3. get数据不安全,用户提交的数据用户能看到, post对用户不可见;
4. GET请求传输数据量很小, 而POST请求可以传输大量的数据
5. POST传输数据可以通过设置编码的方式正确转化中文, 而GET请求传输的数据没有变化

### 3.1 发起GET请求

1. **简单请求**

- 不带参数

```go
package main

import (
	&#34;fmt&#34;
	&#34;io&#34;
	&#34;io/ioutil&#34;
	&#34;net/http&#34;
)

func main() {
	get, err := http.Get(&#34;http://www.baidu.com&#34;)
	if err != nil {
		fmt.Println(&#34;get error&#34;, err)
		return
	}
	defer func(Body io.ReadCloser) {
		fmt.Println(&#34;close error&#34;)
		err := Body.Close()
		if err != nil {
		}
	}(get.Body)
	all, err := ioutil.ReadAll(get.Body)
	if err != nil {
		fmt.Println(&#34;read error&#34;)
		return 
	}
	fmt.Println(string(all))

}

```

- 带参数

关于GET请求的参数需要使用Go语言内置的`net/url`这个标准库来处理。

```go
func main() {
	apiUrl := &#34;http://127.0.0.1:9090/get&#34;
	// URL param
	data := url.Values{}
	data.Set(&#34;name&#34;, &#34;小王子&#34;)
	data.Set(&#34;age&#34;, &#34;18&#34;)
	u, err := url.ParseRequestURI(apiUrl)
	if err != nil {
		fmt.Printf(&#34;parse url requestUrl failed, err:%v\n&#34;, err)
	}
	u.RawQuery = data.Encode() // URL encode
	fmt.Println(u.String())
	resp, err := http.Get(u.String())
	if err != nil {
		fmt.Printf(&#34;post failed, err:%v\n&#34;, err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf(&#34;get resp failed, err:%v\n&#34;, err)
		return
	}
	fmt.Println(string(b))
}
```

对应的Server端HandlerFunc如下：

```go
func getHandler(w http.ResponseWriter, r *http.Request) {
	defer r.Body.Close()
	data := r.URL.Query()
	fmt.Println(data.Get(&#34;name&#34;))
	fmt.Println(data.Get(&#34;age&#34;))
	answer := `{&#34;status&#34;: &#34;ok&#34;}`
	w.Write([]byte(answer))
}
```

2. **自定义get 请求**

```go
package main

import (
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net/http&#34;
)

func main() {
	client := &amp;http.Client{}
	request, err := http.NewRequest(&#34;GET&#34;, &#34;http://www.baidu.com&#34;, nil)
	if err != nil {
		fmt.Println(err)
	}
	response, err := client.Do(request)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(response.StatusCode) // 响应码

	res, err := ioutil.ReadAll(response.Body)

	fmt.Println(string(res))  // 获取的代码

}

```

### 3.2 发起Post请求

1. **简单post请求**

```go
package main

import (
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net/http&#34;
	&#34;strings&#34;
)

// net/http post demo

func main() {
	url := &#34;http://127.0.0.1:9090/post&#34;
	// 表单数据
	//contentType := &#34;application/x-www-form-urlencoded&#34;
	//data := &#34;name=小王子&amp;age=18&#34;
	// json
	contentType := &#34;application/json&#34;
	data := `{&#34;name&#34;:&#34;小王子&#34;,&#34;age&#34;:18}`
	resp, err := http.Post(url, contentType, strings.NewReader(data))
	if err != nil {
		fmt.Printf(&#34;post failed, err:%v\n&#34;, err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf(&#34;get resp failed, err:%v\n&#34;, err)
		return
	}
	fmt.Println(string(b))
}
```

对应的Server端HandlerFunc如下：

```go
func postHandler(w http.ResponseWriter, r *http.Request) {
	defer r.Body.Close()
	// 1. 请求类型是application/x-www-form-urlencoded时解析form数据
	r.ParseForm()
	fmt.Println(r.PostForm) // 打印form数据
	fmt.Println(r.PostForm.Get(&#34;name&#34;), r.PostForm.Get(&#34;age&#34;))
	// 2. 请求类型是application/json时从r.Body读取数据
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		fmt.Printf(&#34;read request.Body failed, err:%v\n&#34;, err)
		return
	}
	fmt.Println(string(b))
	answer := `{&#34;status&#34;: &#34;ok&#34;}`
	w.Write([]byte(answer))
}
```

2. **自定义post请求**

```go
package main

import (
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net/http&#34;
	&#34;strings&#34;
)

func main() {
	resq, err:= http.Post(&#34;http://www.baidu.com&#34;,
		&#34;application/x-www-form-urlencoded&#34;,
		strings.NewReader(&#34;user=admin&amp;pass=admin&#34;))
	if err !=nil{
		fmt.Println(err)
	}
	defer resq.Body.Close()
	body, err:= ioutil.ReadAll(resq.Body)
	if err!=nil{
		fmt.Println(err)
	}
	fmt.Println(body)

}
```

## 4. curl工具

- curl 是一个利用URL语法在命令行工作的文件传输工具, 一般称之为下载工具

- Curl 基础命令

  ```go
  
  curl URL // get请求获取页面, 如果是一个图片,将会下载图片
  curl -o URL // 下载到本地
  curl -i URL // 显示头信息
  curl -v URL // 显示一次http通信的整个过程
  curl -X POST --data &#34;data=xxx&#34; URL  //发送post请求
  curl --user-agent &#34;[USER AGENT]&#34; URL  // 添加useragent
  curl --cokie &#34;cookie&#34; URL //添加cookie
  curl --head &#34;head&#34; URL  //添加请求头
  
  ```

## 5. 启动server服务

- 方式一

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;math/rand&#34;
  	&#34;net/http&#34;
  	&#34;time&#34;
  )
  
  func indexHandler(w http.ResponseWriter, r *http.Request) {
  	rand.Seed(time.Now().UnixNano())
  	randNum := rand.Intn(2)
  	if randNum == 0 {
  		time.Sleep(5 * time.Second)
  	}
  	fmt.Fprintf(w, &#34;quick response&#34;)
  }
  
  func main() {
  	http.HandleFunc(&#34;/&#34;, indexHandler)
  	err := http.ListenAndServe(&#34;:8000&#34;, nil)
  	if err != nil {
  		panic(err)
  	}
  }
  ```

  

- 方式二:

  ```go
  package main
  
  import (
  	&#34;log&#34;
  	&#34;net/http&#34;
  	&#34;time&#34;
  )
  
  const ADDR = &#34;127.0.0.1:8080&#34;
  
  func sayBay(w http.ResponseWriter, r *http.Request) {
  	time.Sleep(time.Second * 1)
  	_, err := w.Write([]byte(&#34;bye bye!&#34;))
  	if err != nil {
  		return
  	}
  }
  
  func main() {
  	// 创建路由器
  	mux := http.NewServeMux()
  	// 设置路由规则
  	mux.HandleFunc(&#34;/bye&#34;, sayBay)
  	// 创建服务器
  	server := &amp;http.Server{
  		Addr:         ADDR,
  		WriteTimeout: time.Second * 3,
  		Handler:      mux,
  	}
  	// 监听端口并提供服务
  	log.Println(&#34;starting http server at: &#34; &#43; ADDR)
  	log.Fatal(server.ListenAndServe())
  }
  ```

  

## 常见错误码

- 200 成功状态吗
- 301 临时重定向
- 302 永久重定向
- 400 客户端 语法错误
- 401 客户端身份认证失败
- 403 拒绝访问
- 404 找不到资源
- 500 服务器内部错误
- 501 服务器不支持
- 502 错误的网关
- 503 服务器过载
- 504 网关超时
- 505 http协议错误



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515175131/  

