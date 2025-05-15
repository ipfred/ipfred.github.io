# 8-2.sqlx的使用



&gt;  在项目中我们通常可能会使用`database/sql`连接MySQL数据库。本文借助使用`sqlx`实现批量插入数据的例子，介绍了`sqlx`中可能被你忽视了的`sqlx.In`和`DB.NamedExec`方法。

# 一. 基本操作

## 1. sqlx介绍

- 在项目中我们通常可能会使用`database/sql`连接MySQL数据库。`sqlx`可以认为是Go语言内置`database/sql`的超集，它在优秀的内置`database/sql`基础上提供了一组扩展。这些扩展中除了大家常用来查询的`Get(dest interface{}, ...) error`和`Select(dest interface{}, ...) error`外还有很多其他强大的功能。

## 2. 安装sqlx

```go
go get github.com/jmoiron/sqlx

// 拉取master分支最新代码
go get github.com/jmoiron/sqlx@master
```

## 3. 连接数据库

```go
package main

import (
	&#34;fmt&#34;
	&#34;github.com/jmoiron/sqlx&#34;
)

var DbSqlx *sqlx.DB

func InitDbSqlx() (err error) {
    	  //用户:密码@tcp(ip:端口)/数据库?charset=utf8mb4&amp;parseTime=True
	dsn := &#34;root:@tcp(127.0.0.1:3306)/go_mysql_test?charset=utf8mb4&amp;parseTime=True&#34;
	// 也可以使用MustConnect连接不成功就panic
	DbSqlx, err = sqlx.Connect(&#34;mysql&#34;, dsn)
	if err != nil {
		fmt.Printf(&#34;connect DB failed, err:%v\n&#34;, err)
		return
	}
	DbSqlx.SetMaxOpenConns(20)
	DbSqlx.SetMaxIdleConns(10)
	return
}
func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
    
    
}
```

## 4. 查询

查询单行数据和查询多行数据示例代码如下：

```go
package main

import &#34;fmt&#34;


// 查询单条数据示例
func QueryRowSqlxDemo(id int) {
	sqlStr := &#34;select id, name, age from user where id=?&#34;
	var u User
	err := DbSqlx.Get(&amp;u, sqlStr, id)
	if err != nil {
		fmt.Printf(&#34;get failed, err:%v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;id:%d name:%s age:%d\n&#34;, u.Id, u.Name, u.Age)

}

// 查询多条数据示例
func QueryMultiRowSqlxDemo(id int) {
	sqlStr := &#34;select id, name, age from user where id &gt; ?&#34;
	var users []User
	err := DbSqlx.Select(&amp;users, sqlStr, id)
	if err != nil {
		fmt.Printf(&#34;query failed, err:%v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;users:%#v\n&#34;, users)
}

func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
    // 查询单条数据示例
	QueryRowSqlxDemo(1)
	// 查询多条数据示例
	QueryMultiRowSqlxDemo(1)
    
}
```

## 5. 插入

sqlx中的exec方法与原生sql中的exec使用基本一致：

```go
package main

import &#34;fmt&#34;


// 插入数据示例
func InsertRowSqlxDemo(name string, age int) {
	sqlStr := &#34;insert into user(name, age) values (?,?)&#34;
	ret, err := DbSqlx.Exec(sqlStr, name, age)
	if err != nil {
		fmt.Printf(&#34;insert failed, err:%v\n&#34;, err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf(&#34;get lastinsert ID failed, err:%v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;insert success, the id is %d.\n&#34;, theID)

}
func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
   // 插入数据
	InsertRowSqlxDemo(&#34;RandySunSqlx&#34;, 18)
    
}
```

## 6. **更新**



```go
package main

import &#34;fmt&#34;

// 更新数据示例
func UpdateRowSqlxDemo(id, age int) {
	sqlStr := &#34;update user set age=? where id = ?&#34;
	ret, err := DbSqlx.Exec(sqlStr, age, id)
	if err != nil {
		fmt.Printf(&#34;update failed, err:%v\n&#34;, err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf(&#34;get RowsAffected failed, err:%v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;update success, affected rows:%d\n&#34;, n)

}

func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
  	// 更新数据
	UpdateRowSqlxDemo(7, 20)
    
}
```

## 7. **删除**



```go
package main

import &#34;fmt&#34;


// 删除数据示例
func DeleteRowSqlxDemo(id int) {
	sqlStr := &#34;delete from user where id = ?&#34;
	ret, err := DbSqlx.Exec(sqlStr, id)
	if err != nil {
		fmt.Printf(&#34;delete failed, err:%v\n&#34;, err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf(&#34;get RowsAffected failed, err:%v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;delete success, affected rows:%d\n&#34;, n)
}

func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
   // 删除数据
	DeleteRowSqlxDemo(7)
    
}
```

# 二. 特殊方法

## 1. NamedExec

`DB.NamedExec`方法用来绑定SQL语句与结构体或map中的同名字段。

```go
package main

// 指定map同名字段
func InsertUserSqlxDemo() (err error) {
	sqlStr := &#34;INSERT INTO user (name,age) VALUES (:name,:age)&#34;
	_, err = DbSqlx.NamedExec(sqlStr,
		map[string]interface{}{
			&#34;name&#34;: &#34;RandySun2&#34;,
			&#34;age&#34;:  18,
		})
	return
}
func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
   // 删除数据
	InsertUserSqlxDemo()
    
}
```

## 2. NamedQuery

与`DB.NamedExec`同理，这里是支持查询。

```go
package main

import &#34;fmt&#34;

/*

@create 2021-08-31-8:34
*/
func NamedQuerySqlxDemo() {
	sqlStr := &#34;SELECT * FROM user WHERE name=:name&#34;
	// 使用map做命名查询
	rows, err := DbSqlx.NamedQuery(sqlStr, map[string]interface{}{&#34;name&#34;: &#34;Barry&#34;})
	if err != nil {
		fmt.Printf(&#34;db.NamedQuery failed, err:%v\n&#34;, err)
		return
	}
	defer rows.Close()
	for rows.Next() {
		var u User
        // 放到结构体中
		err := rows.StructScan(&amp;u)
        // 放到map
		//err := rows.MapScan(&amp;u)
		// 放到切片
		//err := rows.SliceScan(&amp;u)
		if err != nil {
			fmt.Printf(&#34;scan failed, err:%v\n&#34;, err)
			continue
		}
		fmt.Printf(&#34;user:%#v\n&#34;, u)
	}

	u := User{
		Name: &#34;Randy&#34;,
	}
	// 使用结构体命名查询，根据结构体字段的 db tag进行映射
	rows, err = DbSqlx.NamedQuery(sqlStr, u)
	if err != nil {
		fmt.Printf(&#34;db.NamedQuery failed, err:%v\n&#34;, err)
		return
	}
	defer rows.Close()
	for rows.Next() {
		var u User
		err := rows.StructScan(&amp;u)
		if err != nil {
			fmt.Printf(&#34;scan failed, err:%v\n&#34;, err)
			continue
		}
		fmt.Printf(&#34;user:%#v\n&#34;, u)
	}
}
func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
   
	NamedQuerySqlxDemo()
    
}
```

## 3. 事务操作

对于事务操作，我们可以使用`sqlx`中提供的`db.Beginx()`和`tx.Exec()`方法。示例代码如下：

```go
package main

import (
	&#34;errors&#34;
	&#34;fmt&#34;
)


// 事务
func TransactionSqlxDemo() (err error) {

	tx, err := DbSqlx.Beginx() // 开启事务
	if err != nil {
		fmt.Printf(&#34;begin trans failed, err:%v\n&#34;, err)
		return err
	}

	defer func() {
		if p := recover(); p != nil {
			tx.Rollback()
			panic(p) // re-throw panic after Rollback
		} else if err != nil {
			fmt.Println(&#34;rollback&#34;)
			tx.Rollback() // err is non-nil; don&#39;t change it
		} else {
			err = tx.Commit() // err is nil; if Commit returns error update err
			fmt.Println(&#34;commit&#34;)
		}
	}()
	sqlStr1 := &#34;Update user set age=20 where id=?&#34;

	rs, err := tx.Exec(sqlStr1, 1)
	if err != nil {
		return err
	}
	n, err := rs.RowsAffected()
	if err != nil {
		return err
	}
	if n != 1 {
		return errors.New(&#34;exec sqlStr1 failed&#34;)
	}
	sqlStr2 := &#34;Update user set age=50 where i=?&#34;
	rs, err = tx.Exec(sqlStr2, 5)
	if err != nil {
		return err
	}
	n, err = rs.RowsAffected()
	if err != nil {
		return err
	}
	if n != 1 {
		return errors.New(&#34;exec sqlStr1 failed&#34;)
	}
	return err

}
func main() {
    // sqlx使用
    err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
   
	TransactionSqlxDemo()
    
}
```

# 三. sqlx.In

&gt;  `sqlx.In`是`sqlx`提供的一个非常方便的函数。

## 0. 表结构

为了方便演示插入数据操作，这里创建一个`user`表，表结构如下：

```sql
CREATE TABLE `user` (
    `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(20) DEFAULT &#39;&#39;,
    `age` INT(11) DEFAULT &#39;0&#39;,
    PRIMARY KEY(`id`)
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

## 0. 结构体

定义一个`user`结构体，字段通过tag与数据库中user表的列一致。

```go
type User struct {
	Name string `db:&#34;name&#34;`
	Age  int    `db:&#34;age&#34;`
}
```

## 1. sqlx.In的批量插入示例

### 1.2 bindvars（绑定变量）

查询占位符`?`在内部称为**bindvars（查询占位符）**,它非常重要。你应该始终使用它们向数据库发送值，因为它们可以防止SQL注入攻击。`database/sql`不尝试对查询文本进行任何验证；它与编码的参数一起按原样发送到服务器。除非驱动程序实现一个特殊的接口，否则在执行之前，查询是在服务器上准备的。因此`bindvars`是特定于数据库的:

- MySQL中使用`?`
- PostgreSQL使用枚举的`$1`、`$2`等bindvar语法
- SQLite中`?`和`$1`的语法都支持
- Oracle中使用`:name`的语法

`bindvars`的一个常见误解是，它们用来在sql语句中插入值。它们其实仅用于参数化，不允许更改SQL语句的结构。例如，使用`bindvars`尝试参数化列或表名将不起作用：

```go
// ？不能用来插入表名（做SQL语句中表名的占位符）
db.Query(&#34;SELECT * FROM ?&#34;, &#34;mytable&#34;)
 
// ？也不能用来插入列名（做SQL语句中列名的占位符）
db.Query(&#34;SELECT ?, ? FROM people&#34;, &#34;name&#34;, &#34;location&#34;)
```

### 1.3 自己拼接语句实现批量插入

比较笨，但是很好理解。就是有多少个User就拼接多少个`(?, ?)`。

```go
// BatchInsertUsers 自行构造批量插入的语句
func BatchInsertUsersSqlxDemo(users []*User) error {
	// 存放 (?, ?) 的slice
	valueStrings := make([]string, 0, len(users))
	// 存放values的slice
	valueArgs := make([]interface{}, 0, len(users)*2)
	// 遍历users准备相关数据
	for _, u := range users {
		// 此处占位符要与插入值的个数对应
		valueStrings = append(valueStrings, &#34;(?, ?)&#34;)
		valueArgs = append(valueArgs, u.Name)
		valueArgs = append(valueArgs, u.Age)
	}
	// 自行拼接要执行的具体语句
	stmt := fmt.Sprintf(&#34;INSERT INTO user (name, age) VALUES %s&#34;,strings.Join(valueStrings, &#34;,&#34;))
	fmt.Println(&#34;手动拼接sql语句sql:&#34;, stmt, valueArgs)
	// 手动拼接sql
	_, err := DbSqlx.Exec(stmt, valueArgs...)
	return err
}

package main

import &#34;fmt&#34;

func main() {
	u1 := User{Name: &#34;RandySun1&#34;, Age: 18}
	u2 := User{Name: &#34;RandySun2&#34;, Age: 28}
	u3 := User{Name: &#34;RandySun3&#34;, Age: 38}
	//
	//// 方法1
	users := []*User{&amp;u1, &amp;u2, &amp;u3}
	err = BatchInsertUsersSqlxDemo(users)
	if err != nil {
		fmt.Printf(&#34;BatchInsertUsers failed, err:%v\n&#34;, err)
	}
}
```

手动拼接sql语句sql: INSERT INTO user (name, age) VALUES (?, ?),(?, ?),(?, ?)

### 1.4 使用sqlx.In实现批量插入

前提是需要我们的结构体实现`driver.Valuer`接口：

```go
// BatchInsertInUsersSqlxDemo 使用sqlx.In帮我们拼接语句和参数, 注意传入的参数是[]interface{}
func (u User) Value() (driver.Value, error) {
	return []interface{}{u.Name, u.Age}, nil
}
```

使用`sqlx.In`实现批量插入代码如下：

```go
package main

import &#34;fmt&#34;

// BatchInsertInUsersSqlxDemo 使用sqlx.In帮我们拼接语句和参数, 注意传入的参数是[]interface{}
func BatchInsertInUsersSqlxDemo(users []interface{}) error {
	query, args, _ := sqlx.In(
		&#34;INSERT INTO user (name, age) VALUES (?), (?), (?)&#34;, // 有几个数据，就要有几个占位符
		users..., // 如果arg实现了 driver.Valuer, sqlx.In 会通过调用 Value()来展开它
	)
	fmt.Println(query) // 查看生成的querystring
	fmt.Println(args)  // 查看生成的args
	_, err := DbSqlx.Exec(query, args...)
	return err
}



func main() {
	u1 := User{Name: &#34;RandySun1&#34;, Age: 18}
	u2 := User{Name: &#34;RandySun2&#34;, Age: 28}
	u3 := User{Name: &#34;RandySun3&#34;, Age: 38}
	// 方法2
	users2 := []interface{}{u1, u2, u3}
	err = BatchInsertInUsersSqlxDemo(users2)
	if err != nil {
		fmt.Printf(&#34;BatchInsertUsers2 failed, err:%v\n&#34;, err)
	}
}
```

### 1.5 使用NamedExec实现批量插入

**注意** ：该功能需1.3.1版本以上，并且1.3.1版本目前还有点问题，sql语句最后不能有空格和`;`

使用`NamedExec`实现批量插入的代码如下：

```go
package main

import &#34;fmt&#34;

// BatchInsertNamedExecUsersSqlxDemo 使用NamedExec实现批量插入
func BatchInsertNamedExecUsersSqlxDemo(users []*User) error {
	_, err := DbSqlx.NamedExec(&#34;INSERT INTO user (name, age) VALUES (:name, :age)&#34;, users)
	return err
}

func main() {
	u1 := User{Name: &#34;RandySun1&#34;, Age: 18}
	u2 := User{Name: &#34;RandySun2&#34;, Age: 28}
	u3 := User{Name: &#34;RandySun3&#34;, Age: 38}
	// 方法3
	users3 := []*User{&amp;u1, &amp;u2, &amp;u3}
	err = BatchInsertNamedExecUsersSqlxDemo(users3)
	if err != nil {
		fmt.Printf(&#34;BatchInsertUsers3 failed, err:%v\n&#34;, err)
	}
}
```

把上面三种方法综合起来试一下：

```go
func main() {
	err := initDB()
	if err != nil {
		panic(err)
	}
	defer DB.Close()
	// 方法1
	users := []*User{&amp;u1, &amp;u2, &amp;u3}
	err = BatchInsertUsersSqlxDemo(users)
	if err != nil {
		fmt.Printf(&#34;BatchInsertUsers failed, err:%v\n&#34;, err)
	}

	// 方法2
	users2 := []interface{}{u1, u2, u3}
	err = BatchInsertInUsersSqlxDemo(users2)
	if err != nil {
		fmt.Printf(&#34;BatchInsertUsers2 failed, err:%v\n&#34;, err)
	}

	// 方法3
	users3 := []*User{&amp;u1, &amp;u2, &amp;u3}
	err = BatchInsertNamedExecUsersSqlxDemo(users3)
	if err != nil {
		fmt.Printf(&#34;BatchInsertUsers3 failed, err:%v\n&#34;, err)
	}
}
```

## 2. sqlx.In的查询示例

关于`sqlx.In`这里再补充一个用法，在`sqlx`查询语句中实现In查询和FIND_IN_SET函数。即实现`SELECT * FROM user WHERE id in (3, 2, 1);`和`SELECT * FROM user WHERE id in (3, 2, 1) ORDER BY FIND_IN_SET(id, &#39;3,2,1&#39;);`。

### 2.1 in查询

查询id在给定id集合中的数据。

```go
package main

import (
	&#34;fmt&#34;
	&#34;github.com/jmoiron/sqlx&#34;
	&#34;strings&#34;
)

// QueryByIds 根据给定ID查询
func QueryByIds(ids []int) (users []User, err error) {
	// 动态填充id
	query, args, err := sqlx.In(&#34;SELECT name, age FROM user WHERE id IN (?)&#34;, ids)
	if err != nil {
		return
	}
	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
	fmt.Println(query, args, err) 
	query = DbSqlx.Rebind(query) // SELECT name, age FROM user WHERE id IN (?, ?, ?)
	fmt.Println(query)
	err = DbSqlx.Select(&amp;users, query, args...)
	return
}
func main() {
    	// sqlx使用
	err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx) //init db success db&amp;{0xc000020ea0 mysql false 0xc00007ea50}
  Ids := []int{3,1,2}
	userList, err := QueryByIds(Ids)
	fmt.Println(userList)  // [{1 18 Randy} {2 30 Jack} {3 200 Barry}]
}

```

### 2.2 in查询和FIND_IN_SET函数

查询id在给定id集合的数据并维持给定id集合的顺序。

```go
package main

import (
	&#34;fmt&#34;
	&#34;github.com/jmoiron/sqlx&#34;
	&#34;strings&#34;
)

// QueryAndOrderByIds 按照指定id查询并维护顺序
func QueryAndOrderByIds(ids []int) (users []User, err error) {
	// 动态填充id
	strIDs := make([]string, 0, len(ids))
	for _, id := range ids {
		strIDs = append(strIDs, fmt.Sprintf(&#34;%d&#34;, id))
	}
	fmt.Println(strIDs)
	query, args, err := sqlx.In(&#34;SELECT id, name, age FROM user WHERE id IN (?) ORDER BY FIND_IN_SET(id, ?)&#34;, ids, strings.Join(strIDs, &#34;,&#34;))
	if err != nil {
		return
	}
	fmt.Println(query, args, err)

	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
	query = DbSqlx.Rebind(query)

	err = DbSqlx.Select(&amp;users, query, args...)
	return
}

func main() {
    	// sqlx使用
	err := InitDbSqlx()
	if err != nil {
		fmt.Printf(&#34;init db failed, err: %v\n&#34;, err)
		return
	}
	fmt.Printf(&#34;init db success db%v\n&#34;, DbSqlx)
   
    Ids := []int{3,1,2}
	userList, err := QueryAndOrderByIds(Ids)
	fmt.Println(userList)
}
```

---

> 作者: [Fred](https://github.com/ipfred)  
> URL: https://ipfred.github.io/lang/go/go_advanced/20250515180204/  

