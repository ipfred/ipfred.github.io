# Java核心基础

# 2 Java核心基础
&gt; 我想和自己说：学习是反复的过程，学习新知识的时候要仔仔细细阅读，错过一个关键词，可能对知识就会有偏差的理解。如果现在没有心情，收藏起来，后面用到的时候再仔细看。希望有帮助。

## 2.0 基本概念先了解
### 类
- 类是Java中的基本编程单元，用于描述对象的属性和行为。通过实例化类，可以创建对象
- 类是封装了数据和方法的结构
- Java中，类概念非常核心和基础，用于组织和构建整个程序。
- 类名和文件名是一致的(后面讲class关键词的时候会细聊)
### 接口
- 接口是一种抽象的类型，它定义了一组方法的签名，但不提供方法的具体实现。
在后面写面向对象章节的时候，会对类和接口进行详解。
### 变量
- 在程序执行过程中，值可以在某个范围内发生改变的量。
- 变量要明确保存数据的数据类型
### 包
- 将相关的类和接口组织在一起，一个包下面可以创建很多类文件和接口文件。直接引用功能包，可能节省代码量。
###  常量
- 在程序执行过程中，值不发生改变的量。
## 2.1 注释
什么是注释，就是对程序进行解释和说明的文字

- 单行注释
	```java
	//单行注释
	```
- 多行注释
```java
	/*
	多行注释
	*/
```
- 文档注释
	
```java
/**
 * 这是一个文档注释示例
 * 它通常包含有关类、方法或字段的详细信息
 */
public class MyClass {
    // 类的成员和方法
}
```
## 2.2 关键字
被Java语言赋予了特殊含义的单词，在idea中会高亮显示，学习完基础知识后，可以再回过头 看看，对每个关键字的用法要熟悉，不然就是没入门
关键字官方文档 [https://docs.oracle.com/javase/tutorial/java/nutsandbolts/_keywords.html](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/_keywords.html)
- 用于定义数据类型的关键字
	- **class** 类的标识 `class HelloWorld {}`, class是Java程序的基本构建块，它包含了数据和方法(死记就行)。Java文件名必须和类名保持一致，为了编译时能够正确识别和定位类(规定，死记就行); 如果一个文件中包含多个类，`只能有一个类声明为 public`，并且文件名必须与public类一致
	```java
	public class MyClass {
		// 主类 文件名必须是 MyClass.java，程序入口点所在的类，必须要用public修饰，为了执行程序的时候，能够被jvm虚拟机访问到
		public static void main(String[] args) {
        // 入口点
    }
	}
	class AnotherClass {// 非公共类，在同一个文件中可以有多个}
	class YetAnotherClass {// 非公共类，建议每个类都存储在独立的文件中，提高代码可读性，如果有多个相关的类，放在同一个包内}
	```
	- **interface** 接口的标识 `interface MyInterface {}`
	- **enum** 枚举 `enum Day { SUNDAY, MONDAY, TUESDAY }`
	- **byte、short、int、long、float、double、char、boolean**(八大基本数据类型)
	- **void** 声明方法没有返回值
- 流程控制关键字
	- if else switch case default while do for break continue return
- 包的关键字
	- `package`：将相关的**类和接口**组织在一起，一个包下面可以创建很多类文件和接口文件，在Java代码中 声明包 `package com.example.myapp` 包名通常是反转的域名，看到包就知道是什么功能
	- `import` 导包，引入其他包中的类 `import com.otherpackage.OtherClass`
- 访问权限修饰符关键字
	&gt; 访问控制权限在Java中为了管理类，方法，变量等成员在其他类中的可见性和访问权限。有利于控制代码的封装性，安全性和可维护性
	
	- `private` 私有的 成员只对声明它的类可见，其他类无法访问
	- `protected` 受保护访问 `同一包内的类和所有子类可见`
	- `public`: 成员对所有类可见，其他类可以自由访问
	- *扩展：default（package-private）默认访问级别，没有修饰符，成员对统一包内的类可见*
- 类，函数，变量修饰符关键字
	- `abstract`用于声明抽象类、抽象方法。抽象类不能被实例化，通常包含抽象方法，子类需要实现这些抽象方法。
	&gt; 什么是抽象类和抽象方法
	
	- `final` 用于声明不可被继承的类 `final class FinalClass{}`;声明不可被重写的方法 `final void finalMethod() {}`;声明不可被修改的变量 `final int constantValue = 10;`
	- `static`表名具有静态属性。
	```java
	// 静态字段属于类，所有类的实例共享相同的静态字段。
	// 通过类名直接访问
	class Myclass {
		static int staticField = 10;
		// 静态方法属于类，不用创建实例，通过类名直接访问
		static void staticMethod() {
		
		}
		// 静态块 用于类加载时执行一些初始化操作
		static {// 静态块}
	}
	```
	- `synchronized` 修饰方法或代码块，确保同一时刻只有同一个线程可以访问修饰的部分
	
- 类与类之间关系关键字
	- `extends`：用于类之间的继承关系
	- `implements`：接口的实现
- 实例相关关键字
	- new 创建对象实例 `ClassName obj = new ClassName();`
	- this：代表当前对象的引用，在类的方法中使用 `this.variable`指的是当前对象的成员变量
	- super：用于调用父类的方法或访问父类的成员
	- instanceof：用于检查对象是否是特定类的实例 `if (obj instanceof ClassName)`
- 异常处理关键字
	- try、catch、finally、throw、throws

- 其他修饰符关键字
	- native strictfp transient volatile 
	- assert：断言检查 `assert age &gt;= 18 : &#34;必须年满18岁&#34;;`
## 2.3 标识符
标识符就是用来给类，接口，方法，变量，包等其名字的规则
- `类、接口` : 大驼峰命名法,第一个单词大写的是类\接口 `HelloWorld, VariableDemo`
- `变量, 方法`: 小驼峰命名法 第一个单词小写的是方法和变量 `zhangSanAge, studentName`
- `常量`：所有字母都大写
- `包`：所有字母全部都小写 `com.baidu`

## 2.4 数据类型
Java是强类型语言，每一个数据都给出了明确的数据类型

-  基本数据类型(简称: 基本类型) 
	- byte, short, char, int, long, float, double, boolean
	- 定义long类型的数据时, 数据后边要加字母L(大小写均可), 建议加L
	- 定义float类型的数据时, 数据后边要加字母F(大小写均可), 建议加F
- 引用数据类型(简称: 引用类型)
  	- String, 数组, 类, 接口

- 数据类型转换
不同类型的数据之间可能会进行运算，而这些数据取值范围不同，存储方式不同，直接进行运算可能会造成数据损失，所以需要类型转换
	- 自动(隐式)类型转换 将取值范围小的类型自动提升为取值范围大的类型，`byte、short、char--&gt;int--&gt;long--&gt;float--&gt;double`
	- 强制(显式)类型转换 将取值范围大的类型强制转换成取值范围小的类型.

```java
public static void main(String[] args) {
        double doubleValue = 123.456;
        int intValue = (int) doubleValue; // 强制将double转换为int

        System.out.println(&#34;原始double值：&#34; &#43; doubleValue);
        System.out.println(&#34;强制转换后的int值：&#34; &#43; intValue);
        /*原始double值：123.456
		强制转换后的int值：123 */
    }
  ```
  
## 2.5 常量
- 整数常量 `1`
- 小数常量 `3.14`
- 字符常量 `A` `B` `20`(20不是字符，是有两个字符组合成的)
- 字符串常量 `&#34;abcd&#34;`
- 布尔常量 `true` `false`
- 空常量 `null`
## 2.6 变量
- 在程序执行过程中，值可以在某个范围内发生改变的量叫变量
- 变量要有明确的数据类型
- 声明方式
	- 数据类型 变量名=初始化的值; `int a=10;`
	- 先声明，再赋值 `int a; a=10;`
```java
//1. 定义一个类, 类名叫: VariableDemo02
public class VariableDemo02 {
    //2. 定义main方法, 作为程序的主入口.
    public static void main(String[] args) {
        //3. 测试byte类型.
        //3.1 定义一个byte类型的变量, 变量名叫b, 初始化值为10.
        byte b = 10;
        //3.2 将变量b的值打印到控制台上.
        System.out.println(b);

        //4. 测试short类型.
        //4.1 定义一个short类型的变量, 变量名叫s, 初始化值为20.
        short s = 20;
        //4.2 将变量s的值打印到控制台上.
        System.out.println(s);

        //5. 测试char类型.
        //5.1 定义一个char类型的变量, 变量名叫c, 初始化值为&#39;A&#39;.
        char c = &#39;A&#39;;
        //5.2 将变量c的值打印到控制台上.
        System.out.println(c);

        //6. 测试int类型
        int a = 10;
        System.out.println(a);

        //7. 测试long类型, 数据后记得加字母L.
        long lon = 100L;
        System.out.println(lon);

        //8. 测试float类型, 数据后边加字母F.
        float f = 10.3F;
        System.out.println(f);

        //9. 测试double类型.
        double d = 5.21;
        System.out.println(d);

        //10. 测试boolean类型.
        boolean bb = true;
        System.out.println(bb);
    }
}
```
## 2.7 运算符
### 2.7.1分类
- 算术运算符
	-  &#43;, -, *, /, %, &#43;&#43;, --
	- 变量前&#43;&#43; ：变量a自己加1，将加1后的结果赋值给b `b=&#43;&#43;a`
	```java
		public static void main(String[] args) { 
	    int a = 1;    
	    int b = &#43;&#43;a;    
	    System.out.println(a);//计算结果是2    
	    System.out.println(b);//计算结果是2
	}
	```
	- 变量后&#43;&#43; ：变量a先把自己的值1，赋值给变量b
	```java
		public static void main(String[] args) {
	    int a = 1;
	    int b = a&#43;&#43;;
	    System.out.println(a);//计算结果是2
	    System.out.println(b);//计算结果是1
	}
	```
- 赋值运算符
	-  =, &#43;=, -=, *=, /=, %=

- 比较(关系)运算符
	-  ==, !=, &gt;, &gt;=, &lt;, &lt;=
- 逻辑运算符
	- &amp;&amp;(并且), 
	- ||(或者), 
	- !(逻辑非), 
	- ^(逻辑异或) 异同的意思, 相同为false, 不同为true.
	```java
	public class LogicalOperatorsExample {
	    public static void main(String[] args) {
	        // 逻辑与运算符 (&amp;&amp;)
	        boolean condition1 = true;
	        boolean condition2 = false;
	
	        // 如果两个条件都为true，则结果为true，否则为false
	        boolean resultAnd = condition1 &amp;&amp; condition2;
	        System.out.println(&#34;逻辑与运算符 (&amp;&amp;) 结果：&#34; &#43; resultAnd);
	
	        // 逻辑或运算符 (||)
	        // 如果至少一个条件为true，则结果为true，否则为false
	        boolean resultOr = condition1 || condition2;
	        System.out.println(&#34;逻辑或运算符 (||) 结果：&#34; &#43; resultOr);
	
	        // 逻辑非运算符 (!)
	        // 如果条件为true，则结果为false；如果条件为false，则结果为true
	        boolean resultNot = !condition1;
	        System.out.println(&#34;逻辑非运算符 (!) 结果：&#34; &#43; resultNot);
	    }
	}
	```
- 三元(三目)运算符
	-  (关系表达式) ? 表达式1：表达式2； 
	- true执行1表达式，false执行2表达式
	```java
	public class OperatorDemo04 {
	    public static void main(String[] args) {
	        //1. 定义两个int类型的变量a. b, 初始化值分别为10, 20
	        int a = 10, b = 20;
	        //2. 通过三元运算符, 获取变量a和b的最大值.
	        int max = a &lt; b ? b : a;
	        //3. 将结果(最大值)打印到控制台上.
	        System.out.println(max);
	    }
	}
	```

## 2.8 流程控制
### 2.8.1 分支结构
如果我们想某些代码是在满足条件的情况下, 才能被执行, 此时就需要用到选择结构了, 选择结构也叫分支结构, 主要分为以下两种:
- if语句, 主要用于`范围的判断`
- switch.case语句, 主要用于`固定值`的判断.

if分支
```java
public class IfExample {
    public static void main(String[] args) {
        int num = 10;

        if (num &gt; 0) {
            System.out.println(&#34;数字是正数&#34;);
        } else if (num &lt; 0) {
            System.out.println(&#34;数字是负数&#34;);
        } else {
            System.out.println(&#34;数字是零&#34;);
        }
    }
}
```

switch分支
```java
public class SwitchExample {
    public static void main(String[] args) {
        int dayOfWeek = 3;
        String dayName;

        switch (dayOfWeek) {
            case 1:
                dayName = &#34;星期一&#34;;
                break;
            case 2:
                dayName = &#34;星期二&#34;;
                break;
            case 3:
                dayName = &#34;星期三&#34;;
                break;
            case 4:
                dayName = &#34;星期四&#34;;
                break;
            case 5:
                dayName = &#34;星期五&#34;;
                break;
            case 6:
                dayName = &#34;星期六&#34;;
                break;
            case 7:
                dayName = &#34;星期日&#34;;
                break;
            default:
                dayName = &#34;无效的日期&#34;;
                break;
        }
        System.out.println(&#34;今天是：&#34; &#43; dayName);
    }
}
```
switch 分支 case穿透，在switch语句中，如果case的后面不写break，将出现case穿透现象，也就是不会在判断下一个case的值，直接向后运行，直到遇到break，或者整体switch结束。

```java
import java.util.Scanner;

public class SwitchDemo08 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println(&#34;请录入一个月份: &#34;);
        int month = sc.nextInt();
        switch (month) {
            case 12:
            case 1:
            case 2:
                System.out.println(&#34;冬季&#34;);
                break;
            case 3:
            case 4:
            case 5:
                System.out.println(&#34;春季&#34;);
                break;
            case 6:
            case 7:
            case 8:
                System.out.println(&#34;夏季&#34;);
                break;
            case 9:
            case 10:
            case 11:
                System.out.println(&#34;秋季&#34;);
                break;
            default:
                System.out.println(&#34;没有这样的日期&#34;);
                break;
        }
    }
}
}
```

### 2.8.2 循环结构
for循环
```java
for(初始化条件1; 判断条件2; 控制条件3) {
    //循环体4;
}
```
for循环求1-100之前偶数和
```java
public class ForDemo04 {
    public static void main(String[] args) {
        //1. 定义变量sum, 用来记录数据和.
        int sum = 0;
        //2. 通过for循环, 依次获取到1 - 100之间的数字.
        for (int i = 1; i &lt;= 100; i&#43;&#43;) {
            //3. 判断当前遍历到的数字是否是偶数.
            if (i % 2 == 0) {
                //4. 走到这里, 说明是偶数, 累加给变量sum.
                sum &#43;= i;
            }
        }
        //5. 打印结果.
        System.out.println(&#34;1 - 100之间的偶数之和是: &#34; &#43; sum);
    }
}
```

while循环
```java
初始化条件1;
while(判断条件2) {
    //循环体3;
    //控制条件4;
}
```
例子
```java
public class WhileDemo01 {
    public static void main(String[] args) {
        int i = 0;
        while(i &lt; 10) {
            System.out.println(&#34;Hello World!&#34;);
            i&#43;&#43;;
        }
    }
}
```

死循环：永远不结束的循环，循环的判断条件是true
- `for(;;) { }`
- `while(true){}`
- `do {} while(true)`

循环跳转: `break continue`
```java
public class ContinueExample {
    public static void main(String[] args) {
        for (int i = 1; i &lt;= 5; i&#43;&#43;) {
            // 如果i为偶数，跳过本次循环
            if (i % 2 == 0) {
                continue;
            }
            /*if (i==3) {
            	break;
            }*/
            System.out.println(&#34;当前数字是：&#34; &#43; i);
        }
    }
}
```
嵌套循环打印99乘法表
```java
public class ForForDemo08 {
    public static void main(String[] args) {
        for (int i = 1; i &lt;= 9; i&#43;&#43;) {       //外循环控制行
            for (int j = 1; j &lt;= i; j&#43;&#43;) {   //内循环控制列
                //1 * 3 = 3   2 * 3 = 6	 3 * 3 = 9
                System.out.print(j &#43; &#34; * &#34; &#43; i &#43; &#34; = &#34; &#43; i * j &#43; &#34;\t&#34;);     
            }
            System.out.println();           //内循环执行结束, 意味着一行打印完毕, 记得要换行.
        }
    }
}
```



---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/java/20240426133928/  

