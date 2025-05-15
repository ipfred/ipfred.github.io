# 17. go执行外部命令

## 执行外部命令

### 1. 方式一：run

- code

  ```go
  func main() {
  	cmd := exec.Command(&#34;ls&#34;, &#34;-l&#34;, &#34;/var/log/&#34;)
  	err := cmd.Run()
  	if err != nil {
  		log.Fatalf(&#34;cmd.Run() failed with %s\n&#34;, err)
  	}
  }
  ```

  - `Run()` 方法会启动命令并等待命令执行完毕。它会阻塞当前 goroutine 直到命令执行完毕，并返回一个 `error` 对象，该对象表示命令执行的错误信息。如果命令执行成功，`Run()` 方法会返回 `nil`
  - 直接调用 Cmd 对象的 Run 函数，返回的只有成功和失败，获取不到任何输出的结果

### 2. 方式二：start &amp; wait

- code

  ```go
  func main() {
      // 使用 Start() 方法启动命令
      cmd = exec.Command(&#34;ping&#34;, &#34;www.baidu.com&#34;)
      if err := cmd.Start(); err != nil {
          fmt.Println(&#34;Error:&#34;, err)
      }
      if err := cmd.Wait(); err != nil {
          fmt.Println(&#34;Error:&#34;, err)
      }
  }
  ```

- `Start()` 方法会启动命令并立即返回。它不会等待命令执行完毕，而是会在后台异步执行命令。`Start()` 方法返回一个 `error` 对象，该对象表示启动命令的错误信息。如果命令启动成功，`Start()` 方法会返回 `nil`

- 在使用 `Start()` 方法启动命令后，我们可以使用 `Wait()` 方法等待命令执行完毕。`Wait()` 方法会阻塞当前 goroutine 直到命令执行完毕，并返回一个 `error` 对象，该对象表示命令执行的错误信息。如果命令执行成功，`Wait()` 方法会返回 `nil`

## 输出日志

&gt; https://darjun.github.io/2022/11/01/godailylib/osexec/

### 1. 标准输出

- code

  ```go
  func main() {
    cmd := exec.Command(&#34;cal&#34;)
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    err := cmd.Run()
    if err != nil {
      log.Fatalf(&#34;cmd.Run() failed: %v\n&#34;, err)
    }
  }
  ```

  

### 2. 转存文件

- code

  ```go
  func main() {
    f, err := os.OpenFile(&#34;out.txt&#34;, os.O_WRONLY|os.O_CREATE, os.ModePerm)
    if err != nil {
      log.Fatalf(&#34;os.OpenFile() failed: %v\n&#34;, err)
    }
  
    cmd := exec.Command(&#34;cal&#34;)
    cmd.Stdout = f
    cmd.Stderr = f
    err = cmd.Run()
    if err != nil {
      log.Fatalf(&#34;cmd.Run() failed: %v\n&#34;, err)
    }
  }
  ```

  

### 3. 发送到网络

- code

  ```go
  func cal(w http.ResponseWriter, r *http.Request) {
    year := r.URL.Query().Get(&#34;year&#34;)
    month := r.URL.Query().Get(&#34;month&#34;)
  
    cmd := exec.Command(&#34;cal&#34;, month, year)
    cmd.Stdout = w
    cmd.Stderr = w
  
    err := cmd.Run()
    if err != nil {
      log.Fatalf(&#34;cmd.Run() failed: %v\n&#34;, err)
    }
  }
  
  func main() {
    http.HandleFunc(&#34;/cal&#34;, cal)
    http.ListenAndServe(&#34;:8080&#34;, nil)
  }
  ```

  

### 4. 手动捕获

- code

  ```go
  package middleware
  
  import (
  	&#34;bufio&#34;
  	&#34;fmt&#34;
  	&#34;git-biz.qianxin-inc.cn/upming/component/sdk-go-framework.git/log&#34;
  	&#34;io&#34;
  	&#34;os/exec&#34;
  )
  
  type stdType int32
  
  const (
  	stdTypeStdout stdType = iota &#43; 1
  	stdTypeStderr
  )
  
  func forkStdLog(cmd *exec.Cmd) error {
  	// 捕获标准输出
  	stdout, err := cmd.StdoutPipe()
  	if err != nil {
  		return fmt.Errorf(&#34;cmd.StdoutPipe() failed with %v&#34;, err)
  	}
  	go func() {
  		printExecStd(bufio.NewReader(stdout))
  	}()
  
  	// 捕获标准错误
  	stderr, err := cmd.StderrPipe()
  	if err != nil {
  		return fmt.Errorf(&#34;cmd.StderrPipe() failed with %v&#34;, err)
  	}
  	go func() {
  		// printExecStd(bufio.NewReader(stderr), stdTypeStderr)  // TODO 中间件s的输出不标准，后期再处理，需要加上这个参数
  		printExecStd(bufio.NewReader(stderr))
  	}()
  	return nil
  }
  
  func printExecStd(reader *bufio.Reader, std ...stdType) {
  	logger := log.WithField(&#34;[ middleware_s ]&#34;, &#34;printExecStd&#34;)
  	var s stdType
  	if len(std) &gt; 0 {
  		s = std[0]
  	} else {
  		s = stdTypeStdout
  	}
  	outputBytes := make([]byte, 1024)
  	for {
  		n, err := reader.Read(outputBytes) // 获取屏幕的实时输出(并不是按照回车分割)
  		if err != nil {
  			if err == io.EOF {
  				break
  			}
  			logger.Errorf(&#34;read %s failed with %v&#34;, std, err)
  		}
  		output := string(outputBytes[:n])
  		if s == stdTypeStdout {
  			logger.Info(output)
  		} else if s == stdTypeStderr {
  			logger.Error(output)
  		}
  	}
  }
  
  ```

  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180303/  

