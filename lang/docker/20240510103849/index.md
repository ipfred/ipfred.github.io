# docker使用常用命令


### docker命令官方文档

[docker | Docker Docs](https://docs.docker.com/reference/cli/docker/)
### 代理配置

/etc/docker/daemon.json 文件写入下面内容(文件不存在新建)

```json
{
  &#34;registry-mirrors&#34;: [
    &#34;https://hub-mirror.c.163.com&#34;,
    &#34;https://mirror.baidubce.com&#34;
  ]
}
```

重启docker服务
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo service docker stop
sudo service docker start
```


## docker基础命令

### 增

```shell
docker pull 镜像文件名 
#下载docker镜像
docker  run  hello-world   #运行镜像文件,生成docker容器实例,docker run命令,会自动下载不存在的镜像
#容器是随时创建,随时删除的,轻量级,每次docker run 都会生成新的容器记录 
#docker容器进程,如果没有在后台运行的话,就会立即挂掉,  (容器中必须有正在工作的进程)

#运行一个活着的容器进程
docker run -d centos /bin/sh -c &#34;while true;do echo &#39;你个糟老头子,不听课,坏得很&#39;; sleep 1;done&#34;
	#运行镜像  
	-d  后台运行的意思
	 centos  指的是镜像文件名  
	/bin/sh  要在这个容器内运行的命令,指定的解释器 shell解释器 
	-c  指定一段shell代码 
	
#进入容器空间内的命令 
docker exec -it 容器id  /bin/bash  #进入容器空间内 
	-i 交互式命令操作
	-t 开启一个新的终端
    exit 退出虚拟容器
	
#运行一个ubuntu容器
docker run -it ubuntu  

```

### 删
```shell
docker rmi  镜像名字/镜像id   #删除镜像文件 
docker rm  容器id/容器进程名字   #删除容器记录
docker rmi -f  #强制删除镜像文件  
docker  rm  `docker ps -aq`   #批量删除容器记录,只能删除挂掉的容器记录 

```

### 改

```shell
#docker容器进程的启停命令
docker start  容器id 
docker stop  容器id  

```

### 查

```shell
docker search  镜像文件名字   #搜索镜像文件
docker images  #列出当前所有的镜像文件 
docker ps   #列出当前记录正在运行的容器进程 
docker ps -a  #列出所有的容器进程,以及挂掉的  

docker logs 容器id  #查看容器内的日志信息 
docker logs -f  容器id   #检测容器内的日志 

```

### 容器镜像

#### 提交自己开发的dokcer镜像文件
1. docker run -it centos /bin/bash #进入一个纯净的centos容器空间内,此时是最小化安装的系统,没有vim 没有py3
2. 在容器空间内 yum install vim ,然后退出容器
3. 提交容器为新的镜像文件
```shell
#  docker commit 容器进程id    镜像文件的名字 
docker commit 419  s21docker-centos-vim 
```
4. 导出docker镜像
```shell
docker save 镜像文件名  &gt; /opt/s21-centos-vim.tar.gz  

```
5. 导入镜像文件
```shell
docker load &lt;  同事给你发的镜像文件
```
6. 推送镜像只dockerhub
```shell
# 仓库管理 创建dockerhub账号:
# 修改镜像之后保存镜像: 
docker commit b573392cad1a mysql:5.7
# 修改镜像标签: 
docker tag mysql:5.7 bigox/mysql:5.7
# 登录docker: 
docker login
# 推送镜像:
docker push bigox/mysql:5.7
```
### 常用docker命令

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/20240513142833.png)

#### docker容器信息
```shell
##查看docker容器版本
docker version
##查看docker容器信息
docker info
##查看docker容器帮助 记不住docker命令可以经常使用docker --help查看
docker --help

```

#### 镜像的操作
&gt; 对镜像的操作，可使用镜像名、镜像长ID或短ID


```shell
### 镜像查看
docker image list
##只显示镜像ID
docker images -q
##含中间映像层
docker images -qa   

### 镜像搜索
docker search mysql
# 镜像下载
docker pull redis
# 下载仓库里的所有redis镜像
docker pull -a redis

# 镜像删除
docker rmi redis
docker rmi -f redis
# 删除多个镜像
docker rmi -f redis tomcat nginx
# 删除本地多个镜像
docker rmi -f $(docker images -q)

# 构建镜像
##（1）编写dockerfile
cd /docker/dockerfile
vim mycentos
##（2）构建docker镜像
docker build -f /docker/dockerfile/mycentos -t mycentos:1.1

```
### 容器操作

```shell
# 容器启动
##新建并启动容器，参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称
docker run -i -t --name mycentos
##后台启动容器，参数：-d  已守护方式启动容器
docker run -d mycentos

# 查看容器
##查看正在运行的容器
docker ps
##查看正在运行的容器的ID
docker ps -q
##查看正在运行&#43;历史运行过的容器
docker ps -a
##显示运行容器总文件大小
docker ps -s
##显示最近创建容器
docker ps -l
##显示最近创建的3个容器
docker ps -n 3
##不截断输出
docker ps --no-trunc 

# 容器进程
##列出redis容器中运行进程
docker top redis
##查看所有运行容器的进程信息
for i in  `docker ps |grep Up|awk &#39;{print $1}&#39;`;do echo \ &amp;&amp;docker top $i; done

# 容器日志
docker logs rabitmq
##查看redis容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；
docker logs -f -t --tail=20 redis
##查看容器redis从2019年05月21日后的最新10条日志。
docker logs --since=&#34;2019-05-21&#34; --tail=10 redis

# 容器的进入和退出
##使用run方式在创建时进入
docker run -it centos /bin/bash
##关闭容器并退出
exit

##直接进入centos 容器启动命令的终端，不会启动新进程，多个attach连接共享容器屏幕，参数：--sig-proxy=false  确保CTRL-D或CTRL-C不会关闭容器
docker attach --sig-proxy=false centos 
##在 centos 容器中打开新的交互模式终端，可以启动新进程，参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端
docker exec -i -t  centos /bin/bash
##以交互模式在容器中执行命令，结果返回到当前终端屏幕
docker exec -i -t centos ls -l /tmp
##以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
docker exec -d centos  touch cache.txt 

# 容器的停止和删除
##停止一个运行中的容器
docker stop redis
##杀掉一个运行中的容器
docker kill redis
##删除一个已停止的容器
docker rm redis
##删除一个运行中的容器
docker rm -f redis
##删除多个容器
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
## -l 移除容器间的网络连接，连接名为 db
docker rm -l db 
## -v 删除容器，并删除容器挂载的数据卷
docker rm -v redis

# 容器和主机间的数据拷贝
##将rabbitmq容器中的文件copy至本地路径
docker cp rabbitmq:/[container_path] [local_path]
##将主机文件copy至rabbitmq容器
docker cp [local_path] rabbitmq:/[container_path]/
##将主机文件copy至rabbitmq容器，目录重命名为[container_path]（注意与非重命名copy的区别）
docker cp [local_path] rabbitmq:/[container_path]

```

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/docker/20240510103849/  

