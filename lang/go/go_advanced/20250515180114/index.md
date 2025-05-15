# 1-12. go链接参数 ldflags

## 前戏：”链接“基础概念

- 参考文章：http://blog.champbay.com/2019/11/25/%e9%9d%99%e6%80%81%e9%93%be%e6%8e%a5%e5%8a%a8%e6%80%81%e9%93%be%e6%8e%a5%e9%9d%99%e6%80%81%e5%ba%93%e5%85%b1%e4%ba%ab%e5%ba%93%e8%bf%99%e4%ba%9b%e6%a6%82%e5%bf%b5%e7%9a%84%e8%af%a6%e8%a7%a3/

## ldflags 用于链接过程

- 通过命令`go tool dist list`查看go支持的平台：

  ```go
  aix/ppc64, android/386, android/amd64, android/arm, android/arm64, darwin/amd64, darwin/arm64, dragonfly/amd64, freebsd/386, freebsd/amd64, freebsd/arm, freebsd/arm64, illumos/amd64, ios/amd64, ios/arm64, js/wasm, linux/386, linux/amd64, linux/arm, linux/arm64, linux/mips, linux/mips64, linux/mips64le, linux/mipsle, linux/ppc64, linux/ppc64le, linux/riscv64, linux/s390x, netbsd/386, netbsd/amd64, netbsd/arm, netbsd/arm64, openbsd/386, openbsd/amd64, openbsd/arm, openbsd/arm64, openbsd/mips64, plan9/386, plan9/amd64, plan9/arm, solaris/amd64, windows/386, windows/amd64, windows/arm, windows/arm64
  ```

  

- 大佬的文章： https://tonybai.com/2017/06/27/an-intro-about-go-portability/

## 配置go应用和变量

- 参数
  - -w 去掉调试信息
  - -s 去掉符号表
  - -X 注入变量, 编译时赋值

- 使用范围：go install 、go build、go run 、go test

- 编译

  1. &#39;${GO_PATH}/main.go&#39; 

     ```
     package main
     var B int
     func main(){
     	fmt.Println(B)
     }
     ```

  2. 编译 `go build -ldflags &#39;-w -s -X ${GO_PATH}/main.B=100&#39; -o main main.go`

  3. 运行二进制文件输出：100

- 使用变量

  ```go
  Module=github.com/pubgo/xxx
  GOPATH=$(shell go env GOPATH)
  Version=$(shell git tag --sort=committerdate | tail -n 1)
  GoVersion=$(shell go version)
  BuildTime=$(shell date &#34;&#43;%F %T&#34;)
  CommitID=$(shell git rev-parse HEAD)
  LDFLAGS:=-ldflags &#34;-X &#39;github.com/pubgo/xxx/version.GoVersion=${GoVersion}&#39; \
  -X &#39;github.com/pubgo/xxx/version.BuildTime=${BuildTime}&#39; \
  -X &#39;github.com/pubgo/xxx/version.GoPath=${GOPATH}&#39; \
  -X &#39;github.com/pubgo/xxx/version.CommitID=${CommitID}&#39; \
  -X &#39;github.com/pubgo/xxx/version.Module=${Module}&#39; \
  -X &#39;github.com/pubgo/xxx/version.Version=${Version:-v0.0.1}&#39;&#34;
  
  
  // go build ${LDFLAGS} -mod vendor -race -v -o main main.go
  ```

  

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180114/  

