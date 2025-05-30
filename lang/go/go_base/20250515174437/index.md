# 1-1. GoPath和GoMod


# GOPATH

1、为什么要配置GOPATH

配置GOPATH的用意是为了方便项目的部署和构建，以及可以直接使用go get 命令下载第三方的包到自己的项目的src下和相关的执行文件bin目录，和中间文件pkg

src ：项目的源代码

pkg ：编译后的生成文件

bin ： 编译后的可执行文件

如果你只是想单独的写个go代码可以不设置GOPATH

2、结合GoLand来讲解GOPATH

2.1：使用goland创建一个gose项目，（可以不配置GOPATH）

\* 环境变量中我没有配置

![image-20220706221641642](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220706221641.png)



\* 新建gose项目

![image-20210228113900982](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20210228113900982.png)

 

问1：index entire GOPATH:如果你选中那么我就把你在环境变量中配置的GOPATH信息加到你的项目中，没必要，点取消吧，我们如果真的需要也可以在项目配置中在进行设置

\* 打开项目的File——&gt;settings

 ![image-20210228113940306](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/image-20210228113940306.png)

 

问2：Global GOPATH

选则你在环境变量中配置的GOPATH路径

问3：Project GOPATH

项目的GOPATH,最好不好设置Global GOPATH,因为那你的项目将会使用到所用配置到GOPATH的文件

问4：Use GOPATH that`s defined in system environment

如果选中这个，他将使用系统定义的环境变量，并设置到Global GOPATH

问5：Index entire GOPATH:

会将当前项目作为gopath

 ![image-20220706221741165](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220706221741.png)

 

\* 最终的项目结构，也可以使用

![image-20220706221733621](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220706221733.png)

 

# go mod

## 简单操作

- go modules 是 golang 1.11 新加的特性。现在1.12 已经发布了，是时候用起来了。Modules官方定义为：

  &gt; 模块是相关Go包的集合。modules是源代码交换和版本控制的单元。 go命令直接支持使用modules，包括记录和解析对其他模块的依赖性。modules替换旧的基于GOPATH的方法来指定在给定构建中使用哪些源文件。

- 设置go mod和go proxy

```bash
go env -w GOBIN=/Users/youdi/go/bin
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct // 使用七牛云的
```

注意： go env -w会将配置写到  `GOENV=&#34;/Users/youdi/Library/Application Support/go/env&#34;`

下面看下我的配置

```objectivec
GO111MODULE=&#34;on&#34;
GOARCH=&#34;amd64&#34;
GOBIN=&#34;/Users/youdi/go/bin&#34;
GOCACHE=&#34;/Users/youdi/Library/Caches/go-build&#34;
GOENV=&#34;/Users/youdi/Library/Application Support/go/env&#34;
GOEXE=&#34;&#34;
GOFLAGS=&#34;&#34;
GOHOSTARCH=&#34;amd64&#34;
GOHOSTOS=&#34;darwin&#34;
GOINSECURE=&#34;&#34;
GONOPROXY=&#34;&#34;
GONOSUMDB=&#34;&#34;
GOOS=&#34;darwin&#34;
GOPATH=&#34;/Users/youdi/go&#34;
GOPRIVATE=&#34;&#34;
GOPROXY=&#34;https://goproxy.cn,direct&#34;
GOROOT=&#34;/usr/local/go&#34;
GOSUMDB=&#34;off&#34;
GOTMPDIR=&#34;&#34;
GOTOOLDIR=&#34;/usr/local/go/pkg/tool/darwin_amd64&#34;
GCCGO=&#34;gccgo&#34;
AR=&#34;ar&#34;
CC=&#34;clang&#34;
CXX=&#34;clang&#43;&#43;&#34;
CGO_ENABLED=&#34;1&#34;
GOMOD=&#34;/dev/null&#34;
CGO_CFLAGS=&#34;-g -O2&#34;
CGO_CPPFLAGS=&#34;&#34;
CGO_CXXFLAGS=&#34;-g -O2&#34;
CGO_FFLAGS=&#34;-g -O2&#34;
CGO_LDFLAGS=&#34;-g -O2&#34;
PKG_CONFIG=&#34;pkg-config&#34;
GOGCCFLAGS=&#34;-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/8m/v_1j4dgs7rzgqq4p_4_8k_nr0000gn/T/go-build221113671=/tmp/go-build -gno-record-gcc-switches -fno-common&#34;
```

- 我们看一下，我修改的内容

`cat /Users/youdi/Library/Application Support/go/env`

```bash
GO111MODULE=on
GOBIN=/Users/youdi/go/bin
GOPROXY=https://goproxy.cn,direct
GOSUMDB=offGO111MODULE
```

- GO111MODULE 有三个值：off, on和auto（默认值）。
  - GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
  - GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
  -  GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：



```go
当前目录在GOPATH/src之外且该目录包含go.mod文件
当前文件在包含go.mod文件的目录下面。
```

当modules功能启用时，依赖包的存放位置变更为$GOPATH/pkg，允许同一个package多个版本并存，且多个项目可以共享缓存的 module

我们看下目录：

cd /Users/youdi/go/pkg

```css
├── darwin_amd64
│   ├── github.com
│   ├── go.etcd.io
│   ├── golang
│   ├── golang.org
│   ├── gopkg.in
│   ├── quickstart
│   └── uc.a
├── mod
│   ├── cache
│   ├── github.com
│   ├── golang.org
│   ├── google.golang.org
│   └── gopkg.in
└── sumdb
    └── sum.golang.org
```

## go mod命令

- golang 提供了 `go mod`命令来管理包。

- go help mod

```bash
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just &#39;go mod&#39;. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using &#39;go get&#39;.
See &#39;go help modules&#39; for an overview of module functionality.

Usage:

    go mod &lt;command&gt; [arguments]

The commands are:

    download    download modules to local cache
    edit        edit go.mod from tools or scripts
    graph       print module requirement graph
    init        initialize new module in current directory
    tidy        add missing and remove unused modules
    vendor      make vendored copy of dependencies
    verify      verify dependencies have expected content
    why         explain why packages or modules are needed

Use &#34;go help mod &lt;command&gt;&#34; for more information about a command.
```

- go mod 有以下命令：

| 命令     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| download | download modules to local cache(下载依赖包)                  |
| edit     | edit go.mod from tools or scripts（编辑go.mod)               |
| graph    | print module requirement graph (打印模块依赖图)              |
| verify   | initialize new module in current directory（在当前目录初始化mod） |
| tidy     | add missing and remove unused modules(拉取缺少的模块，移除不用的模块) |
| vendor   | make vendored copy of dependencies(将依赖复制到vendor下)     |
| verify   | verify dependencies have expected content (验证依赖是否正确） |
| why      | explain why packages or modules are needed(解释为什么需要依赖) |

比较常用的是 `init`,`tidy`, `edit`

## 使用go mod管理一个新项目

### 1. 初始化项目

可以随便找一个目录创建项目，我使用习惯用IDEA进行创建

```bash
mkdir Gone
cd Gone
go mod init Gone
```

查看一下 go.mod文件

```go
module Gone

go 1.14
```

go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如go get、go build、go mod等修改和维护go.mod文件。

go.mod 提供了module, require、replace和exclude 四个命令

- `module` 语句指定包的名字（路径）
- `require` 语句指定的依赖项模块
- `replace` 语句可以替换依赖项模块
- `exclude` 语句可以忽略依赖项模块

### 2. 添加依赖

- 创建 main.go文件

```go
package main

import (
    &#34;github.com/gin-gonic/gin&#34;
)

func main() {
    r := gin.Default()
    r.GET(&#34;/ping&#34;, func(c *gin.Context) {
        c.JSON(200, gin.H{
            &#34;message&#34;: &#34;pong&#34;,
        })
    })
    r.Run() // listen and serve on 0.0.0.0:8080 (for windows &#34;localhost:8080&#34;)
}
```

- 执行 go run main.go 运行代码会发现 go mod 会自动查找依赖自动下载, 再查看 `go.mod`

```bash
module Gone

go 1.14

require github.com/gin-gonic/gin v1.6.3
```

go module 安装 package 的原則是先拉最新的 release tag，若无tag则拉最新的commit

go 会自动生成一个 go.sum 文件来记录 dependency tree

再次执行脚本 go run main.go发现跳过了检查并安装依赖的步骤。

可以使用命令 go list -m -u all 来检查可以升级的package，使用go get -u need-upgrade-package 升级后会将新的依赖版本更新到go.mod * 也可以使用 go get -u 升级所有依赖

去mod包缓存下看看

```bash
/Users/youdi/go/pkg/mod/github.com/gin-gonic/gin@v1.6.3
```

## go get升级

- 运行 go get -u 将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
- 运行 go get -u=patch 将会升级到最新的修订版本
- 运行 go get package@version 将会升级到指定的版本号version
- 运行go get如果有版本的更改，那么go.mod文件也会更改

- **使用replace替换无法直接获取的package**

  - 由于某些已知的原因，并不是所有的package都能成功下载，比如：golang.org下的包。

  - modules 可以通过在 go.mod 文件中使用 replace 指令替换成github上对应的库，比如：

```go
replace (
    golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a =&gt; github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
)
```

## go mod发布和使用

### Creating a Module

如果你设置好go mod了，那你就可以在任何目录下随便创建

```bash
$mkdir gomodone
$cd gomodone
```

在这个目录下创建一个文件`say.go`

```go
package gomodone

import &#34;fmt&#34; 

// say Hi to someone
func SayHi(name string) string {
   return fmt.Sprintf(&#34;Hi, %s&#34;, name)
}
```

初始化一个 `go.mod`文件



```bash
$ go mod init github.com/jacksonyoudi/gomodone
go: creating new go.mod: module github.com/jacksonyoudi/gomodone
```

查看 go.mod内容如下：



```go
github.com/jacksonyoudi/gomodone
go 1.14
```

下面我们要将这个module发布到github上，然后在另外一个程序使用

```bash
$git init
$vim .gitiiignore
$git commit -am &#34;init&#34;
// github创建对应的repo
$git remote add origin git@github.com:jacksonyoudi/gomodone.git
$git push -u origin master
```

执行完，上面我们就相当于发布完了。

如果有人需要使用，就可以使用

```bash
go get github.com/jacksonyoudi/gomodone
```

这个时候没有加tag，所以，没有版本的控制。默认是v0.0.0后面接上时间和commitid。如下：

```bash
gomodone@v0.0.0-20200517004046-ee882713fd1e
```

官方不建议这样做，没有进行版本控制管理。

### module versioning

使用tag，进行版本控制

#### making a release

```bash
git tag v1.0.0
git push --tags
```

操作完，我们的module就发布了一个v1.0.0的版本了。

推荐在这个状态下，再切出一个分支，用于后续v1.0.0的修复推送,不要直接在master分支修复

```bash
$git checkout -b v1
$git push -u origin v1
```

### use our module

上面已经发布了一个v1.0.0的版本，我们可以在另一个项目中使用，创建一个go的项目

```bash
$mkdir Gone
$cd Gone
$vim main.go
```

```go
package main

import (
    &#34;fmt&#34;
    &#34;github.com/jacksonyoudi/gomodone&#34;
)

func main() {
    fmt.Println(gomodone.SayHi(&#34;Roberto&#34;))
}
```

代码写好了，我们生成 go mod文件

```bash
go mod init Gone
```

上面命令执行完，会生成 go mod文件
 看下mod文件：

```bash
module Gone

go 1.14

require (
    github.com/jacksonyoudi/gomodone v1.0.0
)
```

```bash
$go mod tidy
go: finding module for package github.com/jacksonyoudi/gomodone
go: found github.com/jacksonyoudi/gomodone in github.com/jacksonyoudi/gomodone v1.0.0
```

同时还生成了go.sum, 其中包含软件包的哈希值，以确保我们具有正确的版本和文件。

```bash
github.com/jacksonyoudi/gomodone v1.0.1 h1:jFd&#43;qZlAB0R3zqrC9kwO8IgPrAdayMUS0rSHMDc/uG8=
github.com/jacksonyoudi/gomodone v1.0.1/go.mod h1:XWi&#43;BLbuiuC2YM8Qz4yQzTSPtHt3T3hrlNN2pNlyA94=
github.com/jacksonyoudi/gomodone/v2 v2.0.0 h1:GpzGeXCx/Xv2ueiZJ8hEhFwLu7xjxLBjkOYSmg8Ya/w=
github.com/jacksonyoudi/gomodone/v2 v2.0.0/go.mod h1:L8uFPSZNHoAhpaePWUfKmGinjufYdw9c2i70xtBorSw=
```

这个内容是下面的，需要操作执行的结果

go run main.go就可以运行了

### Making a bugfix release

假如fix一个bug,我们在v1版本上进行修复

修改代码如下：

```go
// say Hi to someone
func SayHi(name string) string {
-       return fmt.Sprintf(&#34;Hi, %s&#34;, name)
&#43;       return fmt.Sprintf(&#34;Hi, %s!&#34;, name)
}
```

修复好，我们开始push

```bash
$ git commit -m &#34;Emphasize our friendliness&#34; say.go
$ git tag v1.0.1
$ git push --tags origin v1
```

#### Updating modules

刚才fix bug，所以要在我们使用项目中更新

这个需要我们手动执行更新module操作

我们通过使用我们的好朋友来做到这一点go get：

- 运行  `go get -u` 以使用最新的  minor  版本或修补程序版本（即它将从1.0.0更新到例如1.0.1，或者，如果可用，则更新为1.1.0）
- 运行  go get -u=patch 以使用最新的  修补程序  版本（即，将更新为1.0.1但不更新  为1.1.0）
- 运行go get package@version 以更新到特定版本（例如[github.com/jacksonyoudi/gomodone@v1.0.1](https://links.jianshu.com/go?to=http%3A%2F%2Fgithub.com%2Fjacksonyoudi%2Fgomodone%40v1.0.1)）

目前module最新的也是v1.0.1

```bash
// 更新最新
$go get -u
$go get -u=patch
//指定包，指定版本
$go get github.com/jacksonyoudi/gomodone@v1.0.1
```

操作完，go.mod文件会修改如下:

```go
module Gone

go 1.14

require (
    github.com/jacksonyoudi/gomodone v1.0.1
)
```

#### Major versions

根据语义版本语义，主要版本与次要版本  不同。主要版本可能会破坏向后兼容性。从Go模块的角度来看，主要版本是  完全不同的软件包。乍一看这听起来很奇怪，但这是有道理的：两个不兼容的库版本是两个不同的库。
 比如下面修改，完全破坏了兼容性。

```go
package gomodone

import (
    &#34;errors&#34;
    &#34;fmt&#34;
)

// Hi returns a friendly greeting
// Hi returns a friendly greeting in language lang
func SayHi(name, lang string) (string, error) {
    switch lang {
    case &#34;en&#34;:
        return fmt.Sprintf(&#34;Hi, %s!&#34;, name), nil
    case &#34;pt&#34;:
        return fmt.Sprintf(&#34;Oi, %s!&#34;, name), nil
    case &#34;es&#34;:
        return fmt.Sprintf(&#34;¡Hola, %s!&#34;, name), nil
    case &#34;fr&#34;:
        return fmt.Sprintf(&#34;Bonjour, %s!&#34;, name), nil
    default:
        return &#34;&#34;, errors.New(&#34;unknown language&#34;)
    }
}
```

如上，我们需要不同的大版本，这种情况下

修改 go.mod如下

```go
module github.com/jacksonyoudi/gomodone/v2

go 1.14
```

然后，重新tag，push

```bash
$ git commit say.go -m &#34;Change Hi to allow multilang&#34;
$ git checkout -b v2 # 用于v2版本，后续修复v2
$ git commit go.mod -m &#34;Bump version to v2&#34;
$ git tag v2.0.0
$ git push --tags origin v2 
```

### Updating to a major version

即使发布了库的新不兼容版本，现有软件 也不会中断，因为它将继续使用现有版本1.0.1。go get -u 将不会获得版本2.0.0。
 如果想使用v2.0.0,代码改成如下：



```go
package main

import (
    &#34;fmt&#34;
    &#34;github.com/jacksonyoudi/gomodone/v2&#34;
)

func main() {
    g, err := gomodone.SayHi(&#34;Roberto&#34;, &#34;pt&#34;)
    if err != nil {
        panic(err)
    }
    fmt.Println(g)
}
```

执行 go mod tidy



```bash
go: finding module for package github.com/jacksonyoudi/gomodone/v2
go: downloading github.com/jacksonyoudi/gomodone/v2 v2.0.0
go: found github.com/jacksonyoudi/gomodone/v2 in github.com/jacksonyoudi/gomodone/v2 v2.0.0
```

当然，两个版本都可以同时使用, 使用别名
 如下：



```go
package main

import (
    &#34;fmt&#34;
    &#34;github.com/jacksonyoudi/gomodone&#34;
    mv2 &#34;github.com/jacksonyoudi/gomodone/v2&#34;
)

func main() {
    g, err := mv2.SayHi(&#34;Roberto&#34;, &#34;pt&#34;)
    if err != nil {
        panic(err)
    }
    fmt.Println(g)

    fmt.Println(gomodone.SayHi(&#34;Roberto&#34;))
}
```

执行一下 `go mod tidy`

### Vendoring

默认是忽略vendor的，如果想在项目目录下有vendor可以执行下面命令



```bash
$go vendor
```

当然，如果构建程序的时候，希望使用vendor中的依赖，



```go
$ go build -mod vendor
```

### IDEA下开发GO

1. 创建go项目

1. 创建完项目，会自动生成go mod文件
    如果需要修改，可以手动修改，加入git等操作

2. 写业务逻辑代码

   ![image-20220706221837753](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220706221837.png)

3. 解决依赖，更新go.mod


---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_base/20250515174437/  

