# 11.Go语言设计模式

## 设计原则

1. **单一职责原则**：一个方法只完成一件事，实现高内聚低耦合。
2. **开闭原则**：对扩展开发，对修改关闭（常见做法：多态、基于接口实现、依赖注入）。
3. **里氏替换原则**：子类可以扩展父类的功能，但不能改变父类原有的功能。也就是说：子类继承父类时，除添加新的方法完成新增功能外，尽量不要重写父类的方法。
4. **接口隔离原则**：尽量将臃肿庞大的接口拆分成更小的和更具体的接口；单一职责原则注重的是职责，而接口隔离原则注重的是对接口依赖的隔离。
5. **依赖倒置原则**：高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象；依赖倒置原则是实现开闭原则的重要途径之一，它降低了客户与实现模块之间的耦合。
6. **迪米特原则**： `只与你的直接朋友交谈，不跟“陌生人”说话`，从依赖者的角度来说，只依赖应该依赖的对象。从被依赖者的角度说，只暴露应该暴露的方法。

![img](https://raw.githubusercontent.com/hellolib/pictures/main/Typora/pic-00-gitee/20220723103721.jpeg)

## 设计模式

- 大类：

  1. 创建型模式： 它提供了一种**在创建对象的同时隐藏创建逻辑**的方式，而不是使用 new 运算符直接实例化对象。
  2. 结构性模式
  3. 行为型模式

  | 序号 | 模式 &amp; 描述                                                  | 包括                                                         |
  | :--- | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | 1    | **创建型模式** &lt;br/&gt;这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。 | **工厂模式（**Factory Pattern）&lt;br&gt;抽象工厂模式（Abstract Factory Pattern）&lt;br/&gt;**单例模式**（Singleton Pattern）&lt;br/&gt;**建造者模式**（Builder Pattern）&lt;br/&gt;原型模式（Prototype Pattern） |
  | 2    | **结构型模式** &lt;br/&gt;这些设计模式关注类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式。 | 适配器模式（Adapter Pattern）&lt;br/&gt;桥接模式（Bridge Pattern）&lt;br/&gt;过滤器模式（Filter、Criteria Pattern）&lt;br/&gt;组合模式（Composite Pattern）&lt;br/&gt;**装饰器模式**（Decorator Pattern）&lt;br/&gt;外观模式（Facade Pattern）&lt;br/&gt;享元模式（Flyweight Pattern）&lt;br/&gt;**代理模式**（Proxy Pattern） |
  | 3    | **行为型模式** &lt;br/&gt;这些设计模式特别关注对象之间的通信。     | 责任链模式（Chain of Responsibility Pattern）&lt;br/&gt;命令模式（Command Pattern）&lt;br/&gt;解释器模式（Interpreter Pattern）&lt;br/&gt;迭代器模式（Iterator Pattern）&lt;br/&gt;中介者模式（Mediator Pattern）&lt;br/&gt;备忘录模式（Memento Pattern）**观察者模式**（Observer Pattern）状态模式（State Pattern）空对象模式（Null Object Pattern）**策略模式**（Strategy Pattern）**模板模式**（Template Pattern）**访问者模式**（Visitor Pattern） |

- 举例
  1. 单例模式 once.Do
  1. 工厂模式 : 自定义一个结构体构造方法
  1. options模式: 把函数生成可选参数
  1. 代理模式: 字符串占位符
  1. 生产者消费者模式: 单向channel
  1. 建造者模式: Gorm 大量用了建造者模式， 例如创建一个new方法，生成某个结构体实例





### 1. 单例模式（创建型）

&gt; 单例模式（Singleton Pattern），是**最简单的一个模式**。在Go中，单例模式指的是全局只有一个实例，并且它负责创建自己的对象。单例模式不仅有利于减少内存开支，还有减少系统性能开销、防止多个实例产生冲突等优点。
&gt;
&gt; 因为单例模式保证了实例的全局唯一性，而且只被初始化一次，所以比较适合**全局共享一个实例，且只需要被初始化一次的场景**，例如数据库实例、全局配置、全局任务池等。

#### 1.1 饿汉模式

&gt; 懒汉方式指全局的单例实例在第一次被使用时创建。因为实例是在包被导入时初始化的，所以如果初始化耗时，会导致程序加载时间比较长。

- 实现方式

  ```go
  package singleton
  
  type singleton struct {
  }
  
  var ins *singleton = &amp;singleton{}
  
  func GetInsOr() *singleton {
      return ins
  }
  ```

- 饿汉模式可以将问题及早暴露，懒汉式虽然支持延迟加载，但是这只是把冷启动时间放到了第一次使用的时候，并没有本质上解决问题，并且为了实现懒汉式还不可避免的需要加锁

#### 1.2 懒汉模式

&gt; **懒汉方式真正使用的时候才会创建实例**，运用较广，但它的缺点是非并发安全，在实际使用时需要加锁。

- 代码实现

  ```go
  package singleton
  
  type singleton struct {
  }
  
  var ins *singleton
  
  func GetInsOr() *singleton {
      if ins == nil {
          ins = &amp;singleton{}
      }
      
      return ins
  }
  ```

- 非并发安全，在实际使用时需要加锁。

#### 1.3 双重锁定

&gt; 双重锁检查是在懒汉模式的基础上加锁实现，保证并发安全

- 代码实现

  ```go
  package singleton
  
  import &#34;sync&#34;
  
  type singleton struct {
  }
  
  var ins *singleton
  var mu sync.Mutex
  
  func GetIns() *singleton {
  	if ins == nil {
  		mu.Lock()
  		defer mu.Unlock()
  		ins = &amp;singleton{}
  	}
  	return ins
  }
  ```

- sync.Once 就是基于双重锁定的封装

  - 代码实现

    ```go
    package singleton
    
    import (
        &#34;sync&#34;
    )
    
    type singleton struct {
    }
    
    var ins *singleton
    var once sync.Once
    
    func GetInsOr() *singleton {
        once.Do(func() {
            ins = &amp;singleton{}
        })
        return ins
    }
    ```

  - 实现原理

    ```go
    package sync
    
    import (
    	&#34;sync/atomic&#34;
    )
    
    type Once struct {
    	done uint32
    	m    Mutex
    }
    
    func (o *Once) Do(f func()) {
    	if atomic.LoadUint32(&amp;o.done) == 0 {
    		// Outlined slow-path to allow inlining of the fast-path.
    		o.doSlow(f)
    	}
    }
    
    func (o *Once) doSlow(f func()) {
    	o.m.Lock()
    	defer o.m.Unlock()
    	if o.done == 0 {
    		defer atomic.StoreUint32(&amp;o.done, 1)
    		f()
    	}
    }
    ```

    

### 2. 工厂模式（创建型）

&gt; 工厂模式（Factory Pattern）是面向对象编程中的常用模式。在Go项目开发中，你可以通过使用多种不同的工厂模式，来使代码更简洁明了。Go中的结构体，可以理解为面向对象编程中的类，例如 Person结构体（类）实现了Greet方法。

- 结构体定义

  ```go
  type Person struct {
    Name string
    Age int
  }
  
  func (p Person) Greet() {
    fmt.Printf(&#34;Hi! My name is %s&#34;, p.Name)
  }
  ```

#### 2.1 简单工厂模式

- **简单工厂模式**是最常用、最简单的。它就是一个接受一些参数，然后返回Person实例的函数

  ```go
  type Person struct {
    Name string
    Age int
  }
  
  func (p Person) Greet() {
    fmt.Printf(&#34;Hi! My name is %s&#34;, p.Name)
  }
  
  func NewPerson(name string, age int) *Person {
    return Person{
      Name: name,
      Age: age
    }
  }
  ```

  

#### 2.2 抽象工厂模式

- 和简单工厂模式的唯一区别，就是它返回的是接口而不是结构体。通过返回接口，可以**在你不公开内部实现的情况下，让调用者使用你提供的各种功能**

  ```go
  type Person interface {
    Greet()
  }
  
  type person struct {
    name string
    age int
  }
  
  func (p person) Greet() {
    fmt.Printf(&#34;Hi! My name is %s&#34;, p.name)
  }
  
  // Here, NewPerson returns an interface, and not the person struct itself
  func NewPerson(name string, age int) Person {
    return person{
      name: name,
      age: age
    }
  }
  ```

  

#### 2.3 工厂方法模式

- 在**简单工厂模式**中，依赖于唯一的工厂对象，如果我们需要实例化一个产品，就要向工厂中传入一个参数，获取对应对象；如果要增加一种产品，就要在工厂中修改创建产品的函数。这会导致耦合性过高，这时我们就可以使用**工厂方法模式**。
- 在**工厂方法模式**中，依赖工厂接口，我们可以通过实现工厂接口来创建多种工厂，将对象创建从由一个对象负责所有具体类的实例化，变成由一群子类来负责对具体类的实例化，从而将过程解耦。

- 代码实现

  ```go
  type Person struct {
  	name string
  	age int
  }
  
  func NewPersonFactory(age int) func(name string) Person {
  	return func(name string) Person {
  		return Person{
  			name: name,
  			age: age,
  		}
  	}
  }
  
  newBaby := NewPersonFactory(1)
  baby := newBaby(&#34;john&#34;)
  
  newTeenager := NewPersonFactory(16)
  teen := newTeenager(&#34;jill&#34;)
  ```

  

### 3. 建造者模式（创建型）

- Gorm 大量用了建造者模式， 例如创建一个new方法，生成某个结构体实例

### 4. 策略模式（行为型）

- 策略模式（Strategy Pattern）定义一组算法，将每个算法都封装起来，并且使它们之间可以互换。

- 在什么时候，我们需要用到策略模式呢？

  - 在项目开发中，我们经常要根据不同的场景，采取不同的措施，也就是不同的**策略**。比如，假设我们需要对a、b 这两个整数进行计算，根据条件的不同，需要执行不同的计算方式。我们可以把所有的操作都封装在同一个函数中，然后通过 `if ... else ...` 的形式来调用不同的计算方式，这种方式称之为**硬编码**。
  - 在实际应用中，随着功能和体验的不断增长，我们需要经常添加/修改策略，这样就需要不断修改已有代码，不仅会让这个函数越来越难维护，还可能因为修改带来一些bug。所以为了解耦，需要使用策略模式，定义一些独立的类来封装不同的算法，每一个类封装一个具体的算法（即策略）。

- 代码实现

  ```go
  package strategy
  
  // 策略模式
  
  // 定义一个策略类
  type IStrategy interface {
  	do(int, int) int
  }
  
  // 策略实现：加
  type add struct{}
  
  func (*add) do(a, b int) int {
  	return a &#43; b
  }
  
  // 策略实现：减
  type reduce struct{}
  
  func (*reduce) do(a, b int) int {
  	return a - b
  }
  
  // 具体策略的执行者
  type Operator struct {
  	strategy IStrategy
  }
  
  // 设置策略
  func (operator *Operator) setStrategy(strategy IStrategy) {
  	operator.strategy = strategy
  }
  
  // 调用策略中的方法
  func (operator *Operator) calculate(a, b int) int {
  	return operator.strategy.do(a, b)
  }
  
  //// 在上述代码中，我们定义了策略接口 IStrategy，还定义了 add 和 reduce 两种策略。最后定义了一个策略执行者，可以设置不同的策略，并执行; 我们可以随意更换策略，而不影响Operator的所有实现。
  func TestStrategy(t *testing.T) {
  	operator := Operator{}
  
  	operator.setStrategy(&amp;add{})
  	result := operator.calculate(1, 2)
  	fmt.Println(&#34;add:&#34;, result)
  
  	operator.setStrategy(&amp;reduce{})
  	result = operator.calculate(2, 1)
  	fmt.Println(&#34;reduce:&#34;, result)
  }
  ```

  

### 5. 观察者模式（行为型）

- 观察者模式 (Observer Pattern)，定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知，依赖对象在收到通知后，可自行调用自身的处理程序，实现想要干的事情，比如更新自己的状态。

- 观察者模式也经常被叫做发布 - 订阅（Publish/Subscribe）模式、上面说的定义对象间的一种一对多依赖关系，一 - 指的是发布变更的主体对象，多 - 指的是订阅变更通知的订阅者对象。

- 代码实现

  ```go
  package main
  
  import &#34;fmt&#34;
  
  // Subject 接口，它相当于是发布者的定义
  type Subject interface {
   Subscribe(observer Observer)  // 添加订阅者
   Notify(msg string)  // 发布通知
  }
  
  // Observer 观察者接口 相当于订阅者
  type Observer interface {
   Update(msg string)
  }
  
  // Subject 实现
  type SubjectImpl struct {
   observers []Observer
  }
  
  // Subscribe 添加观察者（订阅者）
  func (sub *SubjectImpl) Subscribe(observer Observer) {
   sub.observers = append(sub.observers, observer)
  }
  
  
  // Notify 发布通知
  func (sub *SubjectImpl) Notify(msg string) {
   for _, o := range sub.observers {
    o.Update(msg)
   }
  }
  
  // Observer1 Observer1
  type Observer1 struct{}
  
  // Update 实现观察者接口
  func (Observer1) Update(msg string) {
   fmt.Printf(&#34;Observer1: %s\n&#34;, msg)
  }
  
  // Observer2 Observer2
  type Observer2 struct{}
  
  // Update 实现观察者接口
  func (Observer2) Update(msg string) {
   fmt.Printf(&#34;Observer2: %s\n&#34;, msg)
  }
  
  func main(){
   sub := &amp;SubjectImpl{}
   sub.Subscribe(&amp;Observer1{})
   sub.Subscribe(&amp;Observer2{})
   sub.Notify(&#34;Hello&#34;)
  }
  ```

  

### 6. 代理模式（结构型）

&gt; 字符串占位符

### 7. 选项模式（行为型）

- 在Python语言中，创建一个对象时，可以给参数设置默认值，这样在不传入任何参数时，可以返回携带默认值的对象，并在需要时修改对象的属性。这种特性可以大大简化开发者创建一个对象的成本，尤其是在对象拥有众多属性时。

- 代码实现

  ```go
  package options
  
  import (
  	&#34;time&#34;
  )
  
  type Connection struct {
  	addr    string
  	cache   bool
  	timeout time.Duration
  }
  
  const (
  	defaultTimeout = 10
  	defaultCaching = false
  )
  
  type options struct {
  	timeout time.Duration
  	caching bool
  }
  
  // Option overrides behavior of Connect.
  type Option interface {
  	apply(*options)
  }
  
  type optionFunc func(*options)
  
  func (f optionFunc) apply(o *options) {
  	f(o)
  }
  
  func WithTimeout(t time.Duration) Option {
  	return optionFunc(func(o *options) {
  		o.timeout = t
  	})
  }
  
  func WithCaching(cache bool) Option {
  	return optionFunc(func(o *options) {
  		o.caching = cache
  	})
  }
  
  // Connect creates a connection.
  func Connect(addr string, opts ...Option) (*Connection, error) {
  	options := options{
  		timeout: defaultTimeout,
  		caching: defaultCaching,
  	}
  
  	for _, o := range opts {
  		o.apply(&amp;options)
  	}
  
  	return &amp;Connection{
  		addr:    addr,
  		cache:   options.caching,
  		timeout: options.timeout,
  	}, nil
  }
  ```

- 另外一种实现默认参数的方法

  ```go
  package options
  
  import (
  	&#34;time&#34;
  )
  
  const (
  	defaultTimeout = 10
  	defaultCaching = false
  )
  
  type Connection struct {
  	addr    string
  	cache   bool
  	timeout time.Duration
  }
  
  // NewConnect creates a connection.
  func NewConnect(addr string) (*Connection, error) {
  	return &amp;Connection{
  		addr:    addr,
  		cache:   defaultCaching,
  		timeout: defaultTimeout,
  	}, nil
  }
  
  // NewConnectWithOptions creates a connection with options.
  func NewConnectWithOptions(addr string, cache bool, timeout time.Duration) (*Connection, error) {
  	return &amp;Connection{
  		addr:    addr,
  		cache:   cache,
  		timeout: timeout,
  	}, nil
  }
  ```

  

### 8. 生产者消费者模式（行为型）

- 单向channel 是经典的生产者消费者模型

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180231/  

