# 

## 网络代理

### 1. 网络代理&amp;网络转发

- 网络代理

  - 用户通过代理请求信息

  - 请求通过网络代理完成转发到达目标服务器

  - 目标服务器相应后再通过网络代理回传给用户
  - 用户不直接连接服务器，网络代理去连接。获取数据后返回给用户

  ![在这里插入图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-02/20200702202736963.png)

- 网络转发

  - 客户端访问公网服务器，数据包在网络上传输时会经过至少一个路由器，对于多个/多层路由，会进行网络转发，让客户端能够访问公网服务器并返回结果。网络传输中是通过IP来确定服务器（主机）的，通过端口来确定应用(或者说进程)，比如微信应用发消息，会有端口号来唯一标识该应用进程。
  - 是路由器对报文的转发操作，中间可能对数据包修改

  ![在这里插入图片描述](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-02/20200702202725777.png)

### 2. 网络代理类型

- **正向代理**：是一种客户端的代理技术, 帮助客户端访问无法访问的代理服务资源, 可以隐藏真是的IP, 比如浏览器的web代理、vpn等；

  &gt; 1. 监听中的代理服务器在接收到客户端的请求后，会创建一个上游的tcp连接，通过回调方法，复制原请求对象，并根据其中的数据配置新的请求中的各种参数
  &gt; 2. 把新请求发送到真实的服务器，并接收到服务器端的返回
  &gt; 3. 代理服务器对响应做一些处理后，返回给客户端

  

- **反向代理**：是一种服务端的代理技术， 帮助服务端做负载均衡、缓存、提供安全校验等，可以隐藏服务器的真实IP。比如lvs技术、nginx反向代理proxy_pass等

  &gt; 1. 代理接收客户端请求，更改请求结构体信息
  &gt; 2. 通过一定的负载均衡算法获取下游服务器地址
  &gt; 3. 把请求发送到下游服务器，并获取返回内容
  &gt; 4. 对返回内容做一些处理，返回给客户端

  ![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-02/reverse-proxy.png)

- **透明代理**：透明代理的意思是客户端根本不需要知道有代理服务器的存在，它改编你的request fields（报文），并会传送真实IP。注意，加密的透明代理则是属于匿名代理，意思是不用设置使用代理了。透明代理实践的例子就是时下很多公司使用的行为管理软件。

  ![image-20240524154116900](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-02/image-20240524154116900.png)

​		

### 3. 特殊请求头

- **Remote Address**: Remote Address 来自 TCP 连接，表示与服务端建立 TCP 连接的设备 IP; 【不可伪造】

- **X-Forwarded-For**: 一个 HTTP 扩展头部

  - X-Forwarded-For 是一个 HTTP 扩展头部。HTTP/1.1（RFC 2616）协议并没有对它的定义，它最开始是由 Squid 这个缓存代理软件引入，用来表示 HTTP 请求端真实 IP。如今它已经成为事实上的标准，被各大 HTTP 代理、负载均衡等转发服务广泛使用，并被写入 RFC 7239（Forwarded HTTP Extension）标准之中。

  - 格式：`X-Forwarded-For: client, proxy1, proxy2`; 

  - 内容由「英文逗号 &#43; 空格」隔开的多个部分组成，最开始的是离服务端最远的设备 IP，然后是每一级代理设备的 IP;最前面的是客户端真实ip；

  - PS：网上有些文章建议这样配置 Nginx，其实并不合理，这样配置之后，安全性确实提高了，但是也导致请求到达 Nginx 之前的所有代理信息都被抹掉，无法为真正使用代理的用户提供更好的服务。还是应该弄明白这中间的原理，具体场景具体分析。

    ```
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    ```

- **X-Real-IP**：客户端实际请求的服务端IP【不可伪造】



## go实现HTTP代理

### 1. 正向代理

- 步骤

  1. 代理接收客户端请求，复制原请求对象，并根据数 据配置新请求各种参数

  2. 把新请求发送到真实服务端，并接收到服务器端返回

  3. 代理服务器对相应做一些处理，然后返回给客户端

- 代码实现

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;io&#34;
  	&#34;net&#34;
  	&#34;net/http&#34;
  	&#34;strings&#34;
  )
  
  type Pxy struct{}
  
  func (p *Pxy) ServeHTTP(rw http.ResponseWriter, req *http.Request) {
  	fmt.Printf(
  		&#34;Received request %s %s %s\n&#34;, req.Method, req.Host, req.RemoteAddr,
  	)
  	transport := http.DefaultTransport
  	// 1. 浅拷贝对象, 然后再新增属性数据
  	outReq := new(http.Request)
  	*outReq = *req
  	if clientIp, _, err := net.SplitHostPort(req.RemoteAddr); err == nil {
  		if prior, ok := outReq.Header[&#34;X-Forwarded-For&#34;]; ok {
  			clientIp = strings.Join(prior, &#34;,&#34;) &#43; &#34;, &#34; &#43; clientIp
  		}
  		outReq.Header.Set(&#34;X-Forwarded-For&#34;, clientIp)
  	}
  	//	2.请求下游
  	res, err := transport.RoundTrip(outReq)
  	if err != nil {
  		rw.WriteHeader(http.StatusBadGateway)
  		return
  	}
  	//	3. 把下游请求内容返回给上游
  	for key, value := range res.Header {
  		for _, v := range value {
  			rw.Header().Add(key, v)
  		}
  
  	}
  	rw.WriteHeader(res.StatusCode)
  	io.Copy(rw, res.Body)
  	res.Body.Close()
  }
  
  func main() {
  	fmt.Println(&#34;server on :8080&#34;)
  	http.Handle(&#34;/&#34;, &amp;Pxy{})
  	err := http.ListenAndServe(&#34;:8080&#34;, nil)
  	if err != nil {
  		panic(err)
  	}
  }
  
  ```

  

### 2. 反向代理

- 简单版反向代理实现

  1. 代理接收客户端请求， 更改请求结构体信息
  2. 通过负载均衡算法获取下游服务地址
  3. 把请求发送到下游服务器，并获取返回内容
  4. 对返回内容做一些处理，然后返回给客户端

- 真是的服务代码实现

  ```go
  /*
   * @date: 2021/12/7
   * @desc: ...
   */
  
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;io&#34;
  	&#34;log&#34;
  	&#34;net/http&#34;
  	&#34;os&#34;
  	&#34;os/signal&#34;
  	&#34;syscall&#34;
  	&#34;time&#34;
  )
  
  type RealServer struct {
  	Addr string
  }
  
  func (r *RealServer) Run() {
  	log.Println(&#34;Starting httpserver at &#34; &#43; r.Addr)
  	mux := http.NewServeMux()
  	mux.HandleFunc(&#34;/&#34;, r.HelloHandler)
  	mux.HandleFunc(&#34;/base/error&#34;, r.ErrorHandler)
  	mux.HandleFunc(&#34;/test_http_string/test_http_string/aaa&#34;, r.TimeoutHandler)
  	server := &amp;http.Server{
  		Addr:         r.Addr,
  		WriteTimeout: time.Second * 3,
  		Handler:      mux,
  	}
  	go func() {
  		log.Fatal(server.ListenAndServe())
  	}()
  }
  
  func (r *RealServer) HelloHandler(w http.ResponseWriter, req *http.Request) {
  	//127.0.0.1:8008/abc?sdsdsa=11
  	//r.Addr=127.0.0.1:8008
  	//req.URL.Path=/abc
  	//fmt.Println(req.Host)
  	upath := fmt.Sprintf(&#34;http://%s%s\n&#34;, r.Addr, req.URL.Path)
  	realIP := fmt.Sprintf(&#34;RemoteAddr=%s,X-Forwarded-For=%v,X-Real-Ip=%v\n&#34;, req.RemoteAddr, req.Header.Get(&#34;X-Forwarded-For&#34;), req.Header.Get(&#34;X-Real-Ip&#34;))
  	header := fmt.Sprintf(&#34;headers =%v\n&#34;, req.Header)
  	io.WriteString(w, upath)
  	io.WriteString(w, realIP)
  	io.WriteString(w, header)
  
  }
  
  func (r *RealServer) ErrorHandler(w http.ResponseWriter, req *http.Request) {
  	upath := &#34;error handler&#34;
  	w.WriteHeader(500)
  	io.WriteString(w, upath)
  }
  
  func (r *RealServer) TimeoutHandler(w http.ResponseWriter, req *http.Request) {
  	time.Sleep(6 * time.Second)
  	upath := &#34;timeout handler&#34;
  	w.WriteHeader(200)
  	io.WriteString(w, upath)
  }
  
  func main() {
  	rs1 := &amp;RealServer{Addr: &#34;127.0.0.1:2003&#34;}
  	rs1.Run()
  	rs2 := &amp;RealServer{Addr: &#34;127.0.0.1:2004&#34;}
  	rs2.Run()
  
  	//监听关闭信号
  	quit := make(chan os.Signal)
  	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
  	&lt;-quit
  }
  
  ```

- 代理服务器代码实现

  ```go
  package main
  
  import (
  	&#34;bufio&#34;
  	&#34;fmt&#34;
  	&#34;log&#34;
  	&#34;net/http&#34;
  	&#34;net/url&#34;
  )
  
  var (
  	proxyAddr = &#34;http://127.0.0.1:2003&#34;
  	port      = &#34;2002&#34;
  )
  
  func handler(rw http.ResponseWriter, req *http.Request) {
  	// 1. 解析代理IP地址, 更改请求体的协议和主机
  	proxy, err := url.Parse(proxyAddr)
  	req.URL.Scheme = proxy.Scheme
  	req.URL.Host = proxy.Host
  	if err != nil {
  		panic(err)
  	}
  
  	// 2. 请求下游
  	transport := http.DefaultTransport
  	resp, err := transport.RoundTrip(req)
  	if err != nil {
  		panic(err)
  	}
  
  	// 3. 把下游请求内容返回给上游
  	for k, vv := range resp.Header {
  		for _, v := range vv {
  			rw.Header().Add(k, v)
  		}
  	}
  	defer resp.Body.Close()
  	bufio.NewReader(resp.Body).WriteTo(rw)
  }
  
  func main() {
  	http.HandleFunc(&#34;/&#34;, handler)
  	log.Println(&#34;server on port &#34; &#43; port)
  	err := http.ListenAndServe(fmt.Sprintf(&#34;:%v&#34;, port), nil)
  	if err != nil {
  		log.Fatal(err)
  	}
  }
  ```



&gt; 上方原生HTTP实现正反向代理,可能存在一下问题:
&gt;
&gt; 1. 没有错误回调及错误日志等处理
&gt; 2. 无法更改代理后返回的内容
&gt; 3. 没有负载均衡
&gt; 4. 没有url重写
&gt; 5. 没有熔断限流,降级,数据统计等功能
&gt;
&gt; 解决以上问题,go在标准库中提供了ReverseProxy实现http代理

## ReverseProxy实现原理

- ReverseProxy在`net/http/httputil/`包下

- ReverseProxy 功能

  1. 提供了4种负载均衡的实现及接口封装,并且支持自定义负载均衡
  2. 通过中间件提供了: 限流, 熔断,降级, 权限,数据统计等功能
  3. 允许更改启动内容
  4. 可以设置错误信息回调
  5. 支持url重写
  6. 支持连接池功能
  7. 支持webSocket
  8. 支持https代理

   

### 1. ReverseProxy 结构

- 结构体详解

  ```go
  type ReverseProxy struct {
     //控制器必须是一个函数，通过该函数内部可以对请求进行修改，比如请求的路径，请求的参数
     Director func(*http.Request)
  
     //连接池，如果为nil，则使用http.DefaultTransport
     Transport http.RoundTripper
  
     //刷新到客户端的刷新间隔,如果拿到一批数据，返回的间隔时间
     FlushInterval time.Duration
  
     //错误记录器
     ErrorLog *log.Logger
  
     //定义一个缓冲池，在复制http响应的时候使用，用以提高请求效率
     BufferPool BufferPool
  
     //修改response返回内容的函数
     //将函数格式定义为以下格式，就能对返回内容进行修改
     ModifyResponse func(*http.Response) error
  
     //以上函数中出错时，会被该方法捕获
     //错误回调函数，如果为nil，则默认为记录提供的错误并返回502状态错误网关响应,
     //当发生异常时(包括整个流程上某一部分发生异常)可以通过该函数进行处理
     ErrorHandler func(http.ResponseWriter, *http.Request, error)
  }
  
  ```

  

### 2. 简单实现反向代理

- 通过httputil下的NewSingleHostReverseProxy()方法可以直接创建一个ReverseProxy
- ReverseProxy实现了Handler接口,所以可以直接当成路由处理器来使用

- 代码实现

  ```go
  import (
  	&#34;log&#34;
  	&#34;net/http&#34;
  	&#34;net/http/httputil&#34;
  	&#34;net/url&#34;
  )
  
  func main() {
  	//1.真实需要访问的地址
  	rs1 := &#34;http://127.0.0.1:9999/base&#34;
  	//通过url.Parse()解析地址
  	url1, err1 := url.Parse(rs1)
  	if err1 != nil {
  		log.Println(err1)
  	}
  	//2.获取到ReverseProxy
  	proxy := httputil.NewSingleHostReverseProxy(url1)
  	//3.ReverseProxy实现了Handler,可以直接当成处理器路由来使用
  	//通过ReverseProxy实现http代理,当访问当前服务8080端口时,
  	//会被ReverseProxy代理到rs1 
  	log.Fatal(http.ListenAndServe(&#34;:8080&#34;, proxy))
  }
  
  //  假设访问当前服务&#34;127.0.0.1:8080/xxx&#34;在经过ReverseProxy代理后,实际会访问到&#34;127.0.0.1:2003/base/xxx&#34;, 内部提供了一定的重写规则
  ```

  



#### 2.1 NewSingleHostReverseProxy() 源码

- NewSingleHostReverseProxy()  是默认提供的单一代理函数;

- 源码

  ```go
  //target url.URL:代理的目标服务,假设为&#34;http://127.0.0.1:2002/base?name=123&#34;
  func NewSingleHostReverseProxy(target *url.URL) *ReverseProxy {
     //1.获取路径参数,根据上面假设的路径,当前targetQuery 就是&#34;name=123&#34;
     targetQuery := target.RawQuery
     
     //2.创建ReverseProxy需要的Director方法
     //Director:用来改写请求路径,请求参数的函数
     director := func(req *http.Request) {
     	  //2.1设置协议Scheme: http
        req.URL.Scheme = target.Scheme 
        //2.2设置主机Host: 127.0.0.1:2002
        req.URL.Host = target.Host 
         //2.3设置path
         //设置规则:比如当前服务到此处的路径为&#34;http://ip:端口号/dir&#34;
         //上面要代理到target指向的path为&#34;/base&#34;
         //拼接后位&#34;/base/dir&#34; 也就是target.path后要拼接当前服务的path
         //joinURLPath()方法中会有一些合并校验等逻辑
        req.URL.Path, req.URL.RawPath = joinURLPath(target, req.URL)
        //2.4 url参数的设置
        if targetQuery == &#34;&#34; || req.URL.RawQuery == &#34;&#34; {
           req.URL.RawQuery = targetQuery &#43; req.URL.RawQuery
        } else {
           req.URL.RawQuery = targetQuery &#43; &#34;&amp;&#34; &#43; req.URL.RawQuery
        }
        //2.4设置请求头
        if _, ok := req.Header[&#34;User-Agent&#34;]; !ok {
           // explicitly disable User-Agent so it&#39;s not set to default value
           req.Header.Set(&#34;User-Agent&#34;, &#34;&#34;)
        }
     }
     //3.创建ReverseProxy设置Director并返回
     return &amp;ReverseProxy{Director: director}
  }
  
  ```

#### 2.2 自定义SingleHostReverseProxy

- NewMyReverseProxy()

  ```go
  package main
  
  import (
  	&#34;bytes&#34;
  	&#34;encoding/json&#34;
  	&#34;errors&#34;
  	&#34;fmt&#34;
  	&#34;io/ioutil&#34;
  	&#34;log&#34;
  	&#34;net/http&#34;
  	&#34;net/http/httputil&#34;
  	&#34;net/url&#34;
  	&#34;regexp&#34;
  	&#34;strings&#34;
  )
  
  // 1.模拟NewSingleHostReverseProxy创建ReverseProxy
  // target: 目标服务
  func NewMySingleHostReverseProxy(target *url.URL) *httputil.ReverseProxy {
  	// 1.获取path上的请求参数
  	targetQuery := target.RawQuery
  
  	// 2.封装用来修改请求路径,请求参数的Director函数
  	director := func(req *http.Request) {
  		// 2.1请求参数
  		re, _ := regexp.Compile(&#34;^/dir(.*)&#34;)
  		req.URL.Path = re.ReplaceAllString(req.URL.Path, &#34;$1&#34;)
  		// 2.2设置协议
  		req.URL.Scheme = target.Scheme
  		// 2.3设置主机地址
  		req.URL.Host = target.Host
  
  		// 2.4 设置path
  		req.URL.Path = singleJoiningSlash(target.Path, req.URL.Path)
  		// 2.5 设置path参数
  		if targetQuery == &#34;&#34; || req.URL.RawQuery == &#34;&#34; {
  			req.URL.RawQuery = targetQuery &#43; req.URL.RawQuery
  		} else {
  			req.URL.RawQuery = targetQuery &#43; &#34;&amp;&#34; &#43; req.URL.RawQuery
  		}
  		// 2.5设置请求头
  		if _, ok := req.Header[&#34;User-Agent&#34;]; !ok {
  			req.Header.Set(&#34;User-Agent&#34;, &#34;&#34;)
  		}
  		// 读取body
  		body, err := ioutil.ReadAll(req.Body)
  		if err != nil {
  			log.Println(&#34;Failed to read request body:&#34;, err)
  			return
  		}
  		fmt.Println(string(body))
  		defer req.Body.Close()
  		data := map[string]int{
  			&#34;apple&#34;:  2,
  			&#34;banana&#34;: 3,
  			&#34;cherry&#34;: 4,
  		}
  		jsonBytes, err := json.Marshal(data)
  		if err != nil {
  			fmt.Println(&#34;Failed to serialize map to JSON:&#34;, err)
  			return
  		}
  		// req.Body = ioutil.NopCloser(bytes.NewReader(body))
  		req.Body = ioutil.NopCloser(bytes.NewReader(jsonBytes))
  		// 注意如果修改Body内容,要同步修改req.ContentLength长度,否则会报错
  		req.ContentLength = int64(len(jsonBytes))
  
  		// 添加请求头
  		req.Header.Set(&#34;token&#34;, &#34;ssss&#34;)
  	}
  
  	// 3.封装可用用来改写响应的modifyFunc 函数
  	modifyFunc := func(res *http.Response) error {
  		if res.StatusCode != 200 {
  			// 3.1此处判断如果响应的http状态码为异常时,封装异常返回
  			return errors.New(&#34;error statusCode&#34;)
  		}
  		// 3.2读取下游服务响应的body
  		oldPayload, err := ioutil.ReadAll(res.Body)
  		if err != nil {
  			return err
  		}
  		// 3.3封装新的响应
  		newPayLoad := []byte(&#34;hello &#34; &#43; string(oldPayload))
  		// 3.4将数据再次填充到resp中(ioutil.NopCloser()该函数直接将byte数据转换为Body中的read)
  		res.Body = ioutil.NopCloser(bytes.NewBuffer(newPayLoad))
  		// 3.5重置响应数据长度
  		res.ContentLength = int64(len(newPayLoad))
  		res.Header.Set(&#34;Content-Length&#34;, fmt.Sprint(len(newPayLoad)))
  		return nil
  	}
  
  	// 4.设置异常回调,在上面几个步骤如果发送异常,返回的err不为nin,
  	// 会执行该函数,执行指定业务逻辑
  	errorHandler := func(res http.ResponseWriter, req *http.Request, err error) {
  		res.Write([]byte(err.Error()))
  	}
  
  	// 5.创建ReverseProxy返回
  	return &amp;httputil.ReverseProxy{Director: director, ModifyResponse: modifyFunc, ErrorHandler: errorHandler}
  }
  
  // 2.启动服务
  func main() {
  	// 2.1代理的目标服务地址,转换为url类型
  	rs1 := &#34;http://127.0.0.1:2003/base&#34;
  	url1, err1 := url.Parse(rs1)
  	if err1 != nil {
  		log.Println(err1)
  	}
  	// 2.2通过自定义的NewSingleHostReverseProxy创建ReverseProxy
  	proxy := NewMySingleHostReverseProxy(url1)
  	// 2.3ReverseProxy实现了ServeHTTP()可以作为Handle使用
  	log.Fatal(http.ListenAndServe(&#34;:8080&#34;, proxy))
  }
  
  // 复制的源码中的
  func singleJoiningSlash(a, b string) string {
  	aslash := strings.HasSuffix(a, &#34;/&#34;)
  	bslash := strings.HasPrefix(b, &#34;/&#34;)
  	switch {
  	case aslash &amp;&amp; bslash:
  		return a &#43; b[1:]
  	case !aslash &amp;&amp; !bslash:
  		return a &#43; &#34;/&#34; &#43; b
  	}
  	return a &#43; b
  }
  
  ```


#### 2.3 高度定制ReverseProxy

- code

  ```go
  func NewMyMultiProxy() *httputil.ReverseProxy {
  	return &amp;httputil.ReverseProxy{
  		Transport: &amp;http.Transport{
  			MaxIdleConns:        200,              // 控制最大连接数
  			MaxIdleConnsPerHost: 200,              // 控制每个主机要保持的最大空闲（保持活动）连接。默认：2。
  			IdleConnTimeout:     90 * time.Second, // 空闲连接超时
  			// 设置超时时间
  			DialContext: (&amp;net.Dialer{
  				Timeout: 10 * time.Second, // 连接超时时间
  			}).DialContext,
  		},
  		Director: func(r *http.Request) {
  			r.URL.Host = &#34;${targetHost}&#34;
  			r.URL.Path = &#34;${targetPath}&#34;
  			r.URL.Scheme = &#34;http&#34;
  		},
  	}
  }
  ```

- 只是定义了部分属性，其他参数和方法可自行定义

### 3. ReverseProxy的ServeHTTP() 源码

- ServerHTTP做了哪些事

  ```go
  验证是否请求终止: 若请求终止,我们就不会把这个服务请求下游，例如关闭浏览器、网络断开等等，那么就会终止请求
  设置请求context信息,如果上游传了部分context信息，那么我就会将这一部分的context信息做设置
  深拷贝header
  修改req: 这里的修改request信息就包含了请求到下游的特殊的head头信息的变更，比如X-Forwarded-For，X-Real-IP
  Upgrade头的特殊处理
  追加ClientIP信息: 这里就是X-Forwarded-For，X-Real-IP这一块的设置
  向下游请求数据: transport、roundtrip？方法
  处理升级协议请求
  移除逐段头部
  修改返回数据
  拷贝头部的数据
  写入状态码
  周期刷新内容到response
  ```

  

- code

  ```go
  func (p *ReverseProxy) ServeHTTP(rw http.ResponseWriter, req *http.Request) {
  	//验证结构体里面有没有设置过ReverseProxy的连接池，没有则使用默认连接池
      transport := p.Transport
  	if transport == nil {
  		transport = http.DefaultTransport 
  	}
  
      //1、验证是否请求终止
       //上下文取得信息，向下转型为CloseNotifier
       //（http.CloseNotifier是一个接口，只有一个方法CloseNotify() &lt;-chan bool，作用是检测连接是否断开）
       //取出里面通知的一个channel，即cn.CloseNotify()，紧接着开启一个协程，一直监听这个channel是否有请求终止的消息，如果有，便执行cancel()方法
  	ctx := req.Context()
      if ctx.Done() != nil {
  	} else if cn, ok := rw.(http.CloseNotifier); ok {
  		var cancel context.CancelFunc
  		ctx, cancel = context.WithCancel(ctx)
  		defer cancel()
  		notifyChan := cn.CloseNotify()
  		go func() {
  			select {
  			case &lt;-notifyChan:
  				cancel()
  			case &lt;-ctx.Done():
  			}
  		}()
  	}
  
      //2、设置context信息
      //通过上游发送过来的req，重新拷贝新建一个outreq对外请求的request，可以理解为往下文请求的一个request
  	outreq := req.Clone(ctx)
       //对outreq的信息做特殊处理
  	if req.ContentLength == 0 {
  		outreq.Body = nil // Issue 16036: nil Body for http.Transport retries
  	}
  	if outreq.Body != nil {
  		defer outreq.Body.Close()
  	}
      
      //3、深拷贝Header
  	if outreq.Header == nil {
  		outreq.Header = make(http.Header) // Issue 33142: historical behavior was to always allocate
  	}
  	
      //4、修改request，也就是之前控制器Director那里，地址和请求信息的修改拼接
  	p.Director(outreq)
      //outreq.Close = false的意思是表示outreq请求到下游的链接是可以被复用的
  	outreq.Close = false
  
      //5、Upgrade头的特殊处理
       //upgradeType(outreq.Header)取出upgrade的类型并判断是否存在
  	reqUpType := upgradeType(outreq.Header)
  	if !ascii.IsPrint(reqUpType) {
  		p.getErrorHandler()(rw, req, fmt.Errorf(&#34;client tried to switch to invalid protocol %q&#34;, reqUpType))
  		return
  	}
       //删除connection的head头信息
  	removeConnectionHeaders(outreq.Header)
  
      //逐段消息头：客户端和第一代理之间的消息头，与是否往下传递head消息头是没有关联的，往下传递的信息中不应该包含这些逐段消息头
  	//删除后端的逐段消息头
  	for _, h := range hopHeaders {
  		outreq.Header.Del(h)
  	}
  
  	//这两个特殊消息头跳过，不进行删除
  	if httpguts.HeaderValuesContainsToken(req.Header[&#34;Te&#34;], &#34;trailers&#34;) {
  		outreq.Header.Set(&#34;Te&#34;, &#34;trailers&#34;)
  	}
  
  	if reqUpType != &#34;&#34; {
  		outreq.Header.Set(&#34;Connection&#34;, &#34;Upgrade&#34;)
  		outreq.Header.Set(&#34;Upgrade&#34;, reqUpType)
  	}
  
      //6、X-Forwarded-For追加ClientIP信息
       //设置 X-Forwarded-For，以逗号&#43;空格分隔
  	if clientIP, _, err := net.SplitHostPort(req.RemoteAddr); err == nil {
  		prior, ok := outreq.Header[&#34;X-Forwarded-For&#34;]
  		omit := ok &amp;&amp; prior == nil // Issue 38079: nil now means don&#39;t populate the header
  		if len(prior) &gt; 0 {
  			clientIP = strings.Join(prior, &#34;, &#34;) &#43; &#34;, &#34; &#43; clientIP
  		}
  		if !omit {
  			outreq.Header.Set(&#34;X-Forwarded-For&#34;, clientIP)
  		}
  	}
  	//7、向下游请求数据，拿到响应response
  	res, err := transport.RoundTrip(outreq)
  	if err != nil {
  		p.getErrorHandler()(rw, outreq, err)
  		return
  	}
  
  	//8、处理升级协议请求
       //验证响应状态码是否为101，是才考虑升级
      // Deal with 101 Switching Protocols responses: (WebSocket, h2c, etc)
  	if res.StatusCode == http.StatusSwitchingProtocols {
  		if !p.modifyResponse(rw, res, outreq) {
  			return
  		}
          //请求升级方法（具体源码步骤见补充）
  		p.handleUpgradeResponse(rw, outreq, res)
  		return
  	}
  
      //9、移除逐段消息头，删除从下游返回的无用的数据
  	removeConnectionHeaders(res.Header)
  
  	for _, h := range hopHeaders {
  		res.Header.Del(h)
  	}
      
      //10、修改response返回内容
  	if !p.modifyResponse(rw, res, outreq) {
  		return
  	}
  
      //11、拷贝头部数据
  	copyHeader(rw.Header(), res.Header)
  
  	 //处理Trailer头部
  	announcedTrailers := len(res.Trailer)
  	if announcedTrailers &gt; 0 {
  		trailerKeys := make([]string, 0, len(res.Trailer))
  		for k := range res.Trailer {
  			trailerKeys = append(trailerKeys, k)
  		}
  		rw.Header().Add(&#34;Trailer&#34;, strings.Join(trailerKeys, &#34;, &#34;))
  	}
  
      //12、写入状态码
  	rw.WriteHeader(res.StatusCode)
  
      //13、按周期刷新内容到response
  	err = p.copyResponse(rw, res.Body, p.flushInterval(res))
  	if err != nil {
  		defer res.Body.Close()
  		if !shouldPanicOnCopyError(req) {
  			p.logf(&#34;suppressing panic for copyResponse error in test; copy error: %v&#34;, err)
  			return
  		}
  		panic(http.ErrAbortHandler)
  	}
      //读取完body内容后，对body进行关闭
  	res.Body.Close()
  
      //对Trailer逻辑处理
  	if len(res.Trailer) &gt; 0 {
  		if fl, ok := rw.(http.Flusher); ok {
  			fl.Flush()
  		}
  	}
  
  	if len(res.Trailer) == announcedTrailers {
  		copyHeader(rw.Header(), res.Trailer)
  		return   
  	}
  
  	for k, vv := range res.Trailer {
  		k = http.TrailerPrefix &#43; k
  		for _, v := range vv {
  			rw.Header().Add(k, v)   
  		}
  	}
  }
  
  ```

  

## ReverseProxy 负载均衡

### 1. 常见负载均衡算法

#### 1.1 随机负载均衡

```go
package load_balance

import (
	&#34;errors&#34;
	&#34;fmt&#34;
	&#34;math/rand&#34;
	&#34;strings&#34;
)

type RandomBalance struct {
	curIndex int
	rss      []string
	//观察主体
	conf LoadBalanceConf
}

func (r *RandomBalance) Add(params ...string) error {
	if len(params) == 0 {
		return errors.New(&#34;param len 1 at least&#34;)
	}
	addr := params[0]
	r.rss = append(r.rss, addr)
	return nil
}

func (r *RandomBalance) Next() string {
	if len(r.rss) == 0 {
		return &#34;&#34;
	}
	r.curIndex = rand.Intn(len(r.rss))
	return r.rss[r.curIndex]
}

func (r *RandomBalance) Get(key string) (string, error) {
	return r.Next(), nil
}

func (r *RandomBalance) SetConf(conf LoadBalanceConf) {
	r.conf = conf
}

func (r *RandomBalance) Update() {
	if conf, ok := r.conf.(*LoadBalanceZkConf); ok {
		fmt.Println(&#34;Update get conf:&#34;, conf.GetConf())
		r.rss = []string{}
		for _, ip := range conf.GetConf() {
			r.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
	if conf, ok := r.conf.(*LoadBalanceCheckConf); ok {
		fmt.Println(&#34;Update get conf:&#34;, conf.GetConf())
		r.rss = nil
		for _, ip := range conf.GetConf() {
			r.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
}

```



#### 1.2 轮询负载均衡

```go
package load_balance

import (
	&#34;errors&#34;
	&#34;fmt&#34;
	&#34;strings&#34;
)

type RoundRobinBalance struct {
	curIndex int
	rss      []string
	//观察主体
	conf LoadBalanceConf
}

func (r *RoundRobinBalance) Add(params ...string) error {
	if len(params) == 0 {
		return errors.New(&#34;param len 1 at least&#34;)
	}
	addr := params[0]
	r.rss = append(r.rss, addr)
	return nil
}

func (r *RoundRobinBalance) Next() string {
	if len(r.rss) == 0 {
		return &#34;&#34;
	}
	lens := len(r.rss) //5
	if r.curIndex &gt;= lens {
		r.curIndex = 0
	}
	curAddr := r.rss[r.curIndex]
	r.curIndex = (r.curIndex &#43; 1) % lens
	return curAddr
}

func (r *RoundRobinBalance) Get(key string) (string, error) {
	return r.Next(), nil
}

func (r *RoundRobinBalance) SetConf(conf LoadBalanceConf) {
	r.conf = conf
}

func (r *RoundRobinBalance) Update() {
	if conf, ok := r.conf.(*LoadBalanceZkConf); ok {
		fmt.Println(&#34;Update get conf:&#34;, conf.GetConf())
		r.rss = []string{}
		for _, ip := range conf.GetConf() {
			r.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
	if conf, ok := r.conf.(*LoadBalanceCheckConf); ok {
		fmt.Println(&#34;Update get conf:&#34;, conf.GetConf())
		r.rss = nil
		for _, ip := range conf.GetConf() {
			r.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
}

```



#### 1.3 加权负载均衡

&gt;1. 参数详解
&gt;
&gt;- **weight**       // 权重值 初始化时对接点约定的权重
&gt;- **currentWeight**   // 节点当前权重  节点临时权重，每轮都会变化
&gt;- **effectiveWeight**  // 有效权重 节点的有效权重，默认与weight相同, 当节点发生一次故障时，name该节点的 effectiveWeight=weight-1 ，
&gt;- **totalWeight**  //所有节点的有效权重之和 sum(effectiveWeight)
&gt;
&gt;2. 算法流程
&gt;
&gt;  a.  currentWeight  =  currentWeight&#43;effectiveWeight
&gt;
&gt;  b.  选中一个最大的currentWeight节点作为选中节点
&gt;
&gt;  c.   选中节点 currentWeight  =  currentWeight - totalWeight（4&#43;3&#43;2=9）
&gt;
&gt;  ![image-20211208152426479](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-02/image-20211208152426479-20211208171645873.png)



```go
package load_balance

import (
	&#34;errors&#34;
	&#34;fmt&#34;
	&#34;strconv&#34;
	&#34;strings&#34;
)

type WeightRoundRobinBalance struct {
	curIndex int
	rss      []*WeightNode
	rsw      []int
	//观察主体
	conf LoadBalanceConf
}

type WeightNode struct {
	addr            string
	weight          int //权重值
	currentWeight   int //节点当前权重
	effectiveWeight int //有效权重
}

func (r *WeightRoundRobinBalance) Add(params ...string) error {
	if len(params) != 2 {
		return errors.New(&#34;param len need 2&#34;)
	}
	parInt, err := strconv.ParseInt(params[1], 10, 64)
	if err != nil {
		return err
	}
	node := &amp;WeightNode{addr: params[0], weight: int(parInt)}
	node.effectiveWeight = node.weight
	r.rss = append(r.rss, node)
	return nil
}

func (r *WeightRoundRobinBalance) Next() string {
	total := 0
	var best *WeightNode
	for i := 0; i &lt; len(r.rss); i&#43;&#43; {
		w := r.rss[i]
		//step 1 统计所有有效权重之和
		total &#43;= w.effectiveWeight

		//step 2 变更节点临时权重为的节点临时权重&#43;节点有效权重
		w.currentWeight &#43;= w.effectiveWeight

		//step 3 有效权重默认与权重相同，通讯异常时-1, 通讯成功&#43;1，直到恢复到weight大小
		if w.effectiveWeight &lt; w.weight {
			w.effectiveWeight&#43;&#43;
		}
		//step 4 选择最大临时权重点节点
		if best == nil || w.currentWeight &gt; best.currentWeight {
			best = w
		}
	}
	if best == nil {
		return &#34;&#34;
	}
	//step 5 变更临时权重为 临时权重-有效权重之和
	best.currentWeight -= total
	return best.addr
}

func (r *WeightRoundRobinBalance) Get(key string) (string, error) {
	return r.Next(), nil
}

func (r *WeightRoundRobinBalance) SetConf(conf LoadBalanceConf) {
	r.conf = conf
}

func (r *WeightRoundRobinBalance) Update() {
	if conf, ok := r.conf.(*LoadBalanceZkConf); ok {
		fmt.Println(&#34;WeightRoundRobinBalance get conf:&#34;, conf.GetConf())
		r.rss = nil
		for _, ip := range conf.GetConf() {
			r.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
	if conf, ok := r.conf.(*LoadBalanceCheckConf); ok {
		fmt.Println(&#34;WeightRoundRobinBalance get conf:&#34;, conf.GetConf())
		r.rss = nil
		for _, ip := range conf.GetConf() {
			r.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
}

```



#### 1.4 一致性hash负载

```go
package load_balance

import (
	&#34;errors&#34;
	&#34;fmt&#34;
	&#34;hash/crc32&#34;
	&#34;sort&#34;
	&#34;strconv&#34;
	&#34;strings&#34;
	&#34;sync&#34;
)

type Hash func(data []byte) uint32

type UInt32Slice []uint32

func (s UInt32Slice) Len() int {
	return len(s)
}

func (s UInt32Slice) Less(i, j int) bool {
	return s[i] &lt; s[j]
}

func (s UInt32Slice) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}

type ConsistentHashBanlance struct {
	mux      sync.RWMutex
	hash     Hash
	replicas int               //复制因子
	keys     UInt32Slice       //已排序的节点hash切片
	hashMap  map[uint32]string //节点哈希和Key的map,键是hash值，值是节点key

	//观察主体
	conf LoadBalanceConf
}

func NewConsistentHashBanlance(replicas int, fn Hash) *ConsistentHashBanlance {
	m := &amp;ConsistentHashBanlance{
		replicas: replicas,
		hash:     fn,
		hashMap:  make(map[uint32]string),
	}
	if m.hash == nil {
		//最多32位,保证是一个2^32-1环
		m.hash = crc32.ChecksumIEEE
	}
	return m
}

// 验证是否为空
func (c *ConsistentHashBanlance) IsEmpty() bool {
	return len(c.keys) == 0
}

// Add 方法用来添加缓存节点，参数为节点key，比如使用IP
func (c *ConsistentHashBanlance) Add(params ...string) error {
	if len(params) == 0 {
		return errors.New(&#34;param len 1 at least&#34;)
	}
	addr := params[0]
	c.mux.Lock()
	defer c.mux.Unlock()
	// 结合复制因子计算所有虚拟节点的hash值，并存入m.keys中，同时在m.hashMap中保存哈希值和key的映射
	for i := 0; i &lt; c.replicas; i&#43;&#43; {
		hash := c.hash([]byte(strconv.Itoa(i) &#43; addr))
		c.keys = append(c.keys, hash)
		c.hashMap[hash] = addr
	}
	// 对所有虚拟节点的哈希值进行排序，方便之后进行二分查找
	sort.Sort(c.keys)
	return nil
}

// Get 方法根据给定的对象获取最靠近它的那个节点
func (c *ConsistentHashBanlance) Get(key string) (string, error) {
	if c.IsEmpty() {
		return &#34;&#34;, errors.New(&#34;node is empty&#34;)
	}
	hash := c.hash([]byte(key))

	// 通过二分查找获取最优节点，第一个&#34;服务器hash&#34;值大于&#34;数据hash&#34;值的就是最优&#34;服务器节点&#34;
	idx := sort.Search(len(c.keys), func(i int) bool { return c.keys[i] &gt;= hash })

	// 如果查找结果 大于 服务器节点哈希数组的最大索引，表示此时该对象哈希值位于最后一个节点之后，那么放入第一个节点中
	if idx == len(c.keys) {
		idx = 0
	}
	c.mux.RLock()
	defer c.mux.RUnlock()
	return c.hashMap[c.keys[idx]], nil
}

func (c *ConsistentHashBanlance) SetConf(conf LoadBalanceConf) {
	c.conf = conf
}

func (c *ConsistentHashBanlance) Update() {
	if conf, ok := c.conf.(*LoadBalanceZkConf); ok {
		fmt.Println(&#34;Update get conf:&#34;, conf.GetConf())
		c.mux.Lock()
		defer c.mux.Unlock()
		c.keys = nil
		c.hashMap = nil
		for _, ip := range conf.GetConf() {
			c.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
	if conf, ok := c.conf.(*LoadBalanceCheckConf); ok {
		fmt.Println(&#34;Update get conf:&#34;, conf.GetConf())
		c.mux.Lock()
		defer c.mux.Unlock()
		c.keys = nil
		c.hashMap = nil
		for _, ip := range conf.GetConf() {
			c.Add(strings.Split(ip, &#34;,&#34;)...)
		}
	}
}


```



### 2. ReverseProxy 集成负载均衡

- main.go

  ```go
  package main
  
  import (
  	&#34;bytes&#34;
  	&#34;io/ioutil&#34;
  	&#34;log&#34;
  	&#34;net&#34;
  	&#34;net/http&#34;
  	&#34;net/http/httputil&#34;
  	&#34;net/url&#34;
  	&#34;picturePro/http/loadBalance&#34;
  	&#34;strconv&#34;
  	&#34;strings&#34;
  	&#34;time&#34;
  )
  
  var (
  	addr      = &#34;127.0.0.1:2002&#34;
  	transport = &amp;http.Transport{
  		DialContext: (&amp;net.Dialer{
  			Timeout:   30 * time.Second, //连接超时
  			KeepAlive: 30 * time.Second, //长连接超时时间
  		}).DialContext,
  		MaxIdleConns:          100,              //最大空闲连接
  		IdleConnTimeout:       90 * time.Second, //空闲超时时间
  		TLSHandshakeTimeout:   10 * time.Second, //tls握手超时时间
  		ExpectContinueTimeout: 1 * time.Second,  //100-continue状态码超时时间
  	}
  )
  
  func NewMultipleHostsReverseProxy(lb loadBalance.LoadBalance) *httputil.ReverseProxy {
  	//请求协调者
  	director := func(req *http.Request) {
  		nextAddr, err := lb.Get(req.RemoteAddr)
  		if err != nil {
  			log.Fatal(&#34;get next addr fail&#34;)
  		}
  		target, err := url.Parse(nextAddr)
  		if err != nil {
  			log.Fatal(err)
  		}
  		targetQuery := target.RawQuery
  		req.URL.Scheme = target.Scheme
  		req.URL.Host = target.Host
  		req.URL.Path = singleJoiningSlash(target.Path, req.URL.Path)
  		if targetQuery == &#34;&#34; || req.URL.RawQuery == &#34;&#34; {
  			req.URL.RawQuery = targetQuery &#43; req.URL.RawQuery
  		} else {
  			req.URL.RawQuery = targetQuery &#43; &#34;&amp;&#34; &#43; req.URL.RawQuery
  		}
  		if _, ok := req.Header[&#34;User-Agent&#34;]; !ok {
  			req.Header.Set(&#34;User-Agent&#34;, &#34;user-agent&#34;)
  		}
  	}
  
  	//更改内容
  	modifyFunc := func(resp *http.Response) error {
  		//请求以下命令：curl &#39;http://127.0.0.1:2002/error&#39;
  		if resp.StatusCode != 200 {
  			//获取内容
  			oldPayload, err := ioutil.ReadAll(resp.Body)
  			if err != nil {
  				return err
  			}
  			//追加内容
  			newPayload := []byte(&#34;StatusCode error:&#34; &#43; string(oldPayload))
  			resp.Body = ioutil.NopCloser(bytes.NewBuffer(newPayload))
  			resp.ContentLength = int64(len(newPayload))
  			resp.Header.Set(&#34;Content-Length&#34;, strconv.FormatInt(int64(len(newPayload)), 10))
  		}
  		return nil
  	}
  
  	//错误回调 ：关闭real_server时测试，错误回调
  	//范围：transport.RoundTrip发生的错误、以及ModifyResponse发生的错误
  	errFunc := func(w http.ResponseWriter, r *http.Request, err error) {
  		//todo 如果是权重的负载则调整临时权重
  		http.Error(w, &#34;ErrorHandler error:&#34;&#43;err.Error(), 500)
  	}
  
  	return &amp;httputil.ReverseProxy{Director: director, Transport: transport, ModifyResponse: modifyFunc, ErrorHandler: errFunc}
  }
  
  func singleJoiningSlash(a, b string) string {
  	aslash := strings.HasSuffix(a, &#34;/&#34;)
  	bslash := strings.HasPrefix(b, &#34;/&#34;)
  	switch {
  	case aslash &amp;&amp; bslash:
  		return a &#43; b[1:]
  	case !aslash &amp;&amp; !bslash:
  		return a &#43; &#34;/&#34; &#43; b
  	}
  	return a &#43; b
  }
  
  func main() {
  	rb := loadBalance.LoadBanlanceFactory(loadBalance.LbRoundRobin)
  	if err := rb.Add(&#34;http://127.0.0.1:2003/base&#34;, &#34;10&#34;); err != nil {
  		log.Println(err)
  	}
  	if err := rb.Add(&#34;http://127.0.0.1:2004/base&#34;, &#34;20&#34;); err != nil {
  		log.Println(err)
  	}
  	proxy := NewMultipleHostsReverseProxy(rb)
  	log.Println(&#34;Starting httpserver at &#34; &#43; addr)
  	log.Fatal(http.ListenAndServe(addr, proxy))
  }
  
  ```

- factory.go

  ```go
  package loadBalance
  
  type LbType int
  
  const (
  	LbRandom LbType = iota
  	LbRoundRobin
  	LbWeightRoundRobin
  	LbConsistentHash
  )
  
  func LoadBanlanceFactory(lbType LbType) LoadBalance {
  	switch lbType {
  	case LbRandom:
  		return &amp;RandomBalance{}
  	case LbConsistentHash:
  		return NewConsistentHashBalance(10, nil)
  	case LbRoundRobin:
  		return &amp;RoundRobinBalance{}
  	case LbWeightRoundRobin:
  		return &amp;WeightRoundRobinBalance{}
  	default:
  		return &amp;RandomBalance{}
  	}
  }
  
  func LoadBanlanceFactorWithConf(lbType LbType, mConf LoadBalanceConf) LoadBalance {
  	//观察者模式
  	switch lbType {
  	case LbRandom:
  		lb := &amp;RandomBalance{}
  		lb.SetConf(mConf)
  		mConf.Attach(lb)
  		lb.Update()
  		return lb
  	case LbConsistentHash:
  		lb := NewConsistentHashBalance(10, nil)
  		lb.SetConf(mConf)
  		mConf.Attach(lb)
  		lb.Update()
  		return lb
  	case LbRoundRobin:
  		lb := &amp;RoundRobinBalance{}
  		lb.SetConf(mConf)
  		mConf.Attach(lb)
  		lb.Update()
  		return lb
  	case LbWeightRoundRobin:
  		lb := &amp;WeightRoundRobinBalance{}
  		lb.SetConf(mConf)
  		mConf.Attach(lb)
  		lb.Update()
  		return lb
  	default:
  		lb := &amp;RandomBalance{}
  		lb.SetConf(mConf)
  		mConf.Attach(lb)
  		lb.Update()
  		return lb
  	}
  }
  
  ```

- Config.go

  ```go
  package loadBalance
  
  import (
  	&#34;fmt&#34;
  	&#34;picturePro/http/loadBalance/zookeeper&#34;
  )
  
  // 配置主题
  type LoadBalanceConf interface {
  	Attach(o Observer)
  	GetConf() []string
  	WatchConf()
  	UpdateConf(conf []string)
  }
  
  type LoadBalanceZkConf struct {
  	observers    []Observer
  	path         string
  	zkHosts      []string
  	confIpWeight map[string]string
  	activeList   []string
  	format       string
  }
  
  func (s *LoadBalanceZkConf) Attach(o Observer) {
  	s.observers = append(s.observers, o)
  }
  
  func (s *LoadBalanceZkConf) NotifyAllObservers() {
  	for _, obs := range s.observers {
  		obs.Update()
  	}
  }
  
  func (s *LoadBalanceZkConf) GetConf() []string {
  	confList := []string{}
  	for _, ip := range s.activeList {
  		weight, ok := s.confIpWeight[ip]
  		if !ok {
  			weight = &#34;50&#34; //默认weight
  		}
  		confList = append(confList, fmt.Sprintf(s.format, ip)&#43;&#34;,&#34;&#43;weight)
  	}
  	return confList
  }
  
  //更新配置时，通知监听者也更新
  func (s *LoadBalanceZkConf) WatchConf() {
  	zkManager := zookeeper.NewZkManager(s.zkHosts)
  	zkManager.GetConnect()
  	fmt.Println(&#34;watchConf&#34;)
  	chanList, chanErr := zkManager.WatchServerListByPath(s.path)
  	go func() {
  		defer zkManager.Close()
  		for {
  			select {
  			case changeErr := &lt;-chanErr:
  				fmt.Println(&#34;changeErr&#34;, changeErr)
  			case changedList := &lt;-chanList:
  				fmt.Println(&#34;watch node changed&#34;)
  				s.UpdateConf(changedList)
  			}
  		}
  	}()
  }
  
  //更新配置时，通知监听者也更新
  func (s *LoadBalanceZkConf) UpdateConf(conf []string) {
  	s.activeList = conf
  	for _, obs := range s.observers {
  		obs.Update()
  	}
  }
  
  func NewLoadBalanceZkConf(format, path string, zkHosts []string, conf map[string]string) (*LoadBalanceZkConf, error) {
  	zkManager := zookeeper.NewZkManager(zkHosts)
  	zkManager.GetConnect()
  	defer zkManager.Close()
  	zlist, err := zkManager.GetServerListByPath(path)
  	if err != nil {
  		return nil, err
  	}
  	mConf := &amp;LoadBalanceZkConf{format: format, activeList: zlist, confIpWeight: conf, zkHosts: zkHosts, path: path}
  	mConf.WatchConf()
  	return mConf, nil
  }
  
  type Observer interface {
  	Update()
  }
  
  type LoadBalanceObserver struct {
  	ModuleConf *LoadBalanceZkConf
  }
  
  func (l *LoadBalanceObserver) Update() {
  	fmt.Println(&#34;Update get conf:&#34;, l.ModuleConf.GetConf())
  }
  
  func NewLoadBalanceObserver(conf *LoadBalanceZkConf) *LoadBalanceObserver {
  	return &amp;LoadBalanceObserver{
  		ModuleConf: conf,
  	}
  }
  
  ```

  



---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/22.go%E4%B8%8Ehttp%E4%BB%A3%E7%90%86/  

