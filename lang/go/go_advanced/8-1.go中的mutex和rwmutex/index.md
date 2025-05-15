# 

&gt;互斥锁（Mutex）: 当一个goroutine获得互斥锁后，其他goroutine无法在使用被上锁的资源，只能等待资源释放
&gt;
&gt;读写锁 (RWMutex): RWMutex是单写多读锁，该锁可以加多个读锁或者一个写锁

## Mutex: 互斥锁

- 结构

  ```go
  type Mutex struct {
      state int32  // 表示互斥锁的状态，比如是否被锁定等。
      sema  uint32 // sema表示信号量，协程阻塞等待该信号量，解锁的协程释放信号量从而唤醒等待信号量的协程。
  }
  ```

- 特点

  - Mutex 是一个互斥锁，可以创建为其他结构体的字段；零值为解锁状态。Mutex 类型的锁和线程无关，可以由不同的线程加锁和解锁。
  - 锁定状态值为1，未锁定状态锁未0
  - **Lock()加锁、Unlock解锁**
  - 当一个goroutine获得互斥锁后，其他goroutine无法在使用被上锁的资源，只能等待资源释放
  - 不可对同一资源重复使用Lock()，必须等待Unlock()后才可再次Lock()，否则将会导致死锁

  ```go
  package main
  
  import (
  	&#34;fmt&#34;
  	&#34;sync&#34;
  )
  
  type UserInfo struct {
  	Name string
  	Age  int
  	sync.Mutex
  }
  
  func (userInfo *UserInfo) updateUserInfo(name string, age int) {
  	userInfo.Lock()
  	defer userInfo.Unlock()
  	userInfo.Name = name
  	userInfo.Age = age
  }
  
  func main() {
  	userInfo := UserInfo{Name: &#34;text1&#34;, Age: 20}
  	fmt.Println(userInfo)
  	userInfo.Lock()
  	fmt.Println(userInfo)
  }
  
  ```

## RWMutex：读写锁

&gt; 1. 写锁需要阻塞写锁：一个协程拥有写锁时，其他协程写锁定需要阻塞
&gt; 2. 写锁需要阻塞读锁：一个协程拥有写锁时，其他协程读锁定需要阻塞
&gt; 3. 读锁需要阻塞写锁：一个协程拥有读锁时，其他协程写锁定需要阻塞
&gt; 4. 读锁不能阻塞读锁：一个协程拥有读锁时，其他协程也可以拥有读锁

- 结构

  ```go
  type RWMutex struct {
  	w           Mutex  // held if there are pending writers
  	writerSem   uint32 // semaphore for writers to wait for completing readers
  	readerSem   uint32 // semaphore for readers to wait for completing writers
  	readerCount int32  // number of pending readers
  	readerWait  int32  // number of departing readers
  }
  ```

- 方法

  - RLock()：读锁定
  - RUnlock()：解除读锁定
  - Lock(): 写锁定，与Mutex完全一致
  - Unlock()：解除写锁定，与Mutex完全一致
  
- 特点

  - RWMutex是单写多读锁，该锁可以加多个读锁或者一个写锁
  - 读锁占用的情况会阻止写，不会阻止读，多个goroutine可以同时获取读锁
  - 写锁会阻止其他gorotine不论读或者写进来，整个锁由写锁goroutine占用
  - 如果在加写锁之前已经有其他的读锁和写锁，则Lock()会阻塞直到该锁可用，为确保该锁可用，已经阻塞的Lock()调用会从获得的锁中排除新的读取器，即写锁权限高于读锁，有写锁时优先进行写锁定
  - RLock()加读锁时，如果存在写锁,则无法加读；当只有读锁或者没有锁时，可以加读锁，读锁可以加载多个
  - Rulock()解读锁，RUnlock撤销RLock()调用，对于其他同时存在的读锁则没有效果
  - 在没有读锁的情况下调用Runlock()会导致panic错误
  
- 使用

  ```go
  type UserInfo struct {
  	Name string
  	sync.RWMutex
  }
  func main() {
  	userInfo := UserInfo{Name: &#34;test1&#34;}
  	go userInfo.userRLock()
  	go userInfo.userRLock()
  	go userInfo.userRLock()
  	go userInfo.userRLock()
  	go userInfo.userLock()
   
  	time.Sleep(time.Second*5)
  }
   
  func (userInfo *UserInfo) userLock() {
  	defer userInfo.Unlock()
  	userInfo.Lock()
  	fmt.Println(&#34;lock&#34;)
  	fmt.Println(&#34;lock&#34;)
  	fmt.Println(&#34;lock&#34;)
  }
   
  func (userInfo *UserInfo) userRLock() {
  	defer userInfo.RUnlock()
  	userInfo.RLock()
  	time.Sleep(time.Second)
  	fmt.Println(&#34;Rlock&#34;)
  	fmt.Println(&#34;Rlock&#34;)
  	fmt.Println(&#34;Rlock&#34;)
  }
  
  
  
  ```

  

---

> 作者:   
> URL: http://localhost:1313/lang/go/go_advanced/8-1.go%E4%B8%AD%E7%9A%84mutex%E5%92%8Crwmutex/  

