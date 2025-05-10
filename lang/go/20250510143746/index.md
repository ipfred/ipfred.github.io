# go开发环境配置


&gt;配置了这么多次go开发环境，每次都踩坑，记录下使用windows系统开发go语言的环境配置，了解go语言的开发环境特点

开发环境：

- VSCode
- go 1.24 多版本管理

## 下载安装

- 官方下载安装
- 建议安装到 默认路径下面 `C:\Program Files\Go`
- 安装完成后，会发现多了些东西

- GOPATH : `%USERPROFILE%\go` 在用户根目录下

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510142946.png)

bin 下面是go的一些插件 用vscode开发的时候 会依赖这些插件
pkg 下面是各个项目下载的第三方包，都会存到这个路径下
- PATH `%USERPROFILE%\go\bin`

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510143203.png)

- 系统环境变量 PATH `C:\Program Files\Go\bin`
这个是下载的go程序的安装路径

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510142741.png)


## 源配置

go第三方包的源在国外，下载速度会比较慢，更换为国内源
推荐使用 **go env** 命令
```shell
# 启用 Go Modules 模式
go env -w GO111MODULE=on

go env -w GOPROXY=https://goproxy.cn,direct
```

&gt; 扩展 go env 命令 配置写到哪个地方了

```shell
$HOME/.config/go/env   # Linux/macOS
%USERPROFILE%\AppData\Roaming\go\env  # Windows
```
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510150238.png)

## 使用vscode开发 go配置

1. 插件市场 搜索 安装 go插件
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510144849.png)

2. 安装 工具 `ctrl &#43; shift &#43; p` 搜索 go install 安装 工具 gopls是必须的

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510145237.png)
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510145322.png)

安装完后插件 也放到了 GOPATH下面

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510145441.png)

&gt; 扩展 介绍下这些插件的作用

| 工具名称         | 版本信息       | 作用描述                                       | 主要用途/使用场景 |
|------------------|----------------|------------------------------------------------|-------------------|
| `gopls`          | `latest`       | Go 语言服务器（Language Server Protocol）     | 提供代码补全、跳转定义、重构等编辑器功能，提升编码效率 |
| `gotests`        | `v1.6.0`       | 自动生成单元测试代码                           | 快速生成函数或方法的测试框架，提高测试覆盖率 |
| `gomodifytags`   | `v1.7.0`       | 修改结构体字段标签（如 JSON、XML、SQL 等）     | 自动添加、删除或修改结构体中的标签，方便序列化处理 |
| `impl`           | `v1.4.0`       | 为接口生成实现模板                             | 快速生成接口的方法骨架，减少手动编写工作量 |
| `goplay`         | `v1.0.0`       | 类似 Go Playground 的本地代码运行环境          | 快速运行小段代码片段，验证逻辑无需创建完整项目 |
| `dlv`            | `latest`       | Go 调试工具（Delve）                          | 设置断点、查看变量、调试程序行为，适合复杂问题排查 |
| `staticcheck`    | `latest`       | 静态代码分析工具（Linter）                    | 检查潜在错误、优化代码质量，遵循最佳实践 |
## 多版本管理
经常会存在一些情况，不同的项目依赖不同的go版本，官方有自己的go 多版本管理方式，挺好用的

[官方go多版本管理](https://go.dev/doc/manage-install) 

```shell
# 先安装一个 go1.10.7 的命令
go install golang.org/dl/go1.10.7@latest
# 再使用这个命令下载 这个版本的go
go1.10.7 download
```
- 第一步 下载 go1.10.7 命令 放在了 GOPATH bin下面

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510144321.png)

- 第二步 使用 go1.10.7 命令下载这个版本的go 存放的位置  当前用户根目录 的 `C:\Users\admin\sdk` 路径下

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510144421.png)

多版本的go 放到哪个地方了呢？

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510144539.png)

后续用VSCode开发的时候，就可以很好的切换到不同的go版本了

也可以通过 **go1.10.7 env GOROOT** 命令查看安装到哪个地方了

### vscode 中切换不同版本的go

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510153554.png)

选择切换成不同的go版本

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510153646.png)

在左侧项目文件夹 也有 go的一些快捷操作

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20250510153742.png)

## go常用的命令

别死记了！ 用的不熟练的时候， **go help**看下

---
相信了解这些细节之后，安装 配置你都能很清楚了

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/20250510143746/  

