# Java开发工具和前言



&gt; 前言：以初学着的身份，准备在该平台整理点最近学习的知识，方便后续查看相关的技术点，有兴趣的可以一块交流学习。目标用最少的文字，理解最多的知识。不说废话，字字珠玑。


## 1.1 简介
- 强类型语言、开源、跨平台、多态、多线程、面向对象
- 完善的异常处理机制，大数据必备的语言
- 1995年出生，父亲 詹姆斯·高斯林(James Gosling)，2009年被Sun公司收购
- 兄弟版本
	- J2SE: 标准版, 也是其他两个版本的基础. 在JDK1.5的时候正式更名为: **JavaSE**.
	- J2ME: 小型版, 一般用来研发嵌入式程序. 已经被Android替代了. 在JDK1.5的时候正式更名为: **JavaME**.
	- J2EE: 企业版, 一般开发企业级互联网程序. 在JDK1.5的时候正式更名为: **JavaEE**
## 1.2 Java环境搭建
- JDK和JRE区别
	- `JRE`: Java运行时环境(Java Runtime Environment) `运行Java`的环境
	- `JDK`: Java development kit Java开发工具包，包含`开发`工具和JRE
	- `JVM`：Java虚拟机(Java Virtual Machine) 和，Java运行环境里要有JVM
- 目录解释
	-  bin: 存放的是编译器和工具
    - db: 存数数据
    -  include: 编译本地方法.
    -  jre: Java运行时文件
    -  lib: 存放类库文件
    -  src.zip: 存放源代码的

    ![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/9d1b56c3518c4b8db6d074c71b574aa5.png)对比这图片看能不能理解每个文件啥意思，不能理解，好好看看上面的介绍
   
   &gt; 怎么安装和配置path，不同的系统自己动手搜搜吧

## 1.3 程序的开发步骤
Java这种静态语言的的开发步骤一般分为
- `编写`：编写源代码，在后缀名为.java的源文件中编写，用idea开发工具
- `编译`:把源代码，编译成计算机能看懂的文件. `javacv执行`生成.class文件
- `执行`:让计算机运行指定的代码程序 `java` 运行

直接整idea开发工具，来个hello world的例子
## 1.4 idea开发工具
- idea 下载地址 [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)
- 激活方式 小程序 `码叔资源` 上找个激活码激活，或淘宝上买一个激活码
- 下载好后一路next安装，建议安装路径放D盘，软件是真大
## 1.5用idea开发一个helloworld
- 新建一个空项目
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/4b98b6567b30487fb8a16303f0411966.png)
效果如下
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/66175214d14a44fa919110a0cf17d192.png)
我这一个空文件夹，怎么和Java关联起来呢，要和JDK关联上，才能用`写Java代码`是吧

- 打开刚刚新建的项目 有个 `项目结构`这个东西后面用到的挺多，配置
	
	![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/25652e557cac41d3b1725d6cf19b8b68.png)
	重点讲讲 project modules libraries的区别
	- `项目`：可以执行你刚刚创建那个空项目的名称，SDK(用Java还是什么开发),编译器输出路径
	- ![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/a53c22d423a84a7995a6701ae5bf583e.png)
	- `模块`：我们新建一个day01的模块，后面可以在这个项目下新增day02 day03的模块，你也可以每天建一个Java项目。
	- ![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/1c9697beee63422d9b9acb10c80f527d.png)
	- 建完之后项目下新增了一个day01的模块，可以在该模块下写代码了(真不容易)
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/09f85e9e011d4523a9f43de411039575.png)![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/d4fe2d7e6a6541048d680f4651b4e91a.png)
	- `删除模块` 删除模块只是这个项目中看不到该模块了，在文件夹中还是存在的
	- ![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/3d36a3df545e4c0eaac1feb28d8b497a.png)



	- `依赖库` ：我这个项目想用第三方的包，咋个办，在这导入就行了，例如连接MySQL的包
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/085dd54b42154714b595f972f66eb177.png)

- 搞了这么久，idea上写代码试下吧,day01模块下 src目录下新建一个 Java源代码文件,输入下面的代码
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/442cd31c4ccf497ca6450a921ebb490e.png)![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/c9b96a220abd441aa0e4577f4bf7f651.png)

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println(&#34;Hello World!&#34;);
    }
}
```
&gt; idea使用小tips，idea很强大，这代码太繁琐了不会写怎么办，idea会自动补全的
&gt; 输入`main` 按tab键，main方法出来了，`sout` 按tab键 print语句出来了，更多idea的技巧有机会，单独出一篇文章

- 运行代码初体验,直接右键run(idea会先编译，然后执行)，终端中会打印出helloworld

![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/dd626b911aa443948910d535babc55c9.png)
![](https://raw.githubusercontent.com/ipfred/my_pics/main/picgo/2404/0d94c8d79feb4c4bb33c71d8a027df8e.png)



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: http://localhost:1313/lang/java/20240426111354/  

