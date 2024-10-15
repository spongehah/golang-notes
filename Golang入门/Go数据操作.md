> 参考：[Golang语言中文文档：#Golang数据操作](https://www.topgoer.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C/)

[TOC]

## Go操作MySQL

### 准备工作

> 准备工作包括：下载第三方库、连接MySQL、新建测试表

将第三方库下载到 GOPATH/pkg/ 路径中

```bash
go get github.com/go-sql-driver/mysql 
go get github.com/jmoiron/sqlx
```

连接MySQL语法：

新建mysql-init.go：

```go
import (
	"fmt"
	"github.com/jmoiron/sqlx"
)

var Db *sqlx.DB

func init() {
	//格式：database, err := sqlx.Open("mysql", "用户名:密码@tcp(地址:端口)/数据库名")
	database, err := sqlx.Open("mysql", "root:123456@tcp(127.0.0.1:3306)/db_test")
	if err != nil {
		fmt.Println("open mysql failed,", err)
		return
	}
	Db = database
}
```

初始化测试表：

```sql
create database db_test;
use db_test;

CREATE TABLE `person` (
    `user_id` int(11) NOT NULL AUTO_INCREMENT,
    `username` varchar(260) DEFAULT NULL,
    `sex` varchar(260) DEFAULT NULL,
    `email` varchar(260) DEFAULT NULL,
    PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

然后在上面mysql-init.go中新建两个结构体struct：需要打上结构体标签，与建表字段一一对应：

```go
type Person struct {
    UserId   int    `db:"user_id"`
    Username string `db:"username"`
    Sex      string `db:"sex"`
    Email    string `db:"email"`
}
```

### 增删改查

- 增删改：Db.Exec()
- 查询：Db.Select()

#### insert

新建operations.go：注意与mysql-init.go是同一个包，这样才可以使用mysql-init.go中的init函数

新建函数Insert：

```go
import (
    "fmt"
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    defer Db.Close()
    Insert()
}

func Insert() {
	result, err := Db.Exec("insert into person(username, sex, email) values (?,?,?)", "小明", "man", "xiaoming@163.com")
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}

	id, err := result.LastInsertId()
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}

	fmt.Println("insert success！the insert id is ", id)
}
```

#### select

在operations.go中新建函数Select：

```go
func main() {
    defer Db.Close()
    //Insert()
    Select()
}

func Select() {
    var person []Person
    
    err := Db.Select(&person, "select * from person")
    if err != nil {
       fmt.Println("select fail, ", err)
       return
    }

    fmt.Println(person)
}
```

#### update

在operations.go中新建函数Update：

```go
func main() {
    defer Db.Close()
    //Insert()
    //Select()
    Update()
}

func Update() {
    res, err := Db.Exec("update person set username = ? where user_id = ?", "Jack", "2")
    if err != nil {
       fmt.Println("exec failed, ", err)
       return
    }

    count, err := res.RowsAffected()
    if err != nil {
       fmt.Println("exec failed, ", err)
       return
    }
    fmt.Println("affected rows: ", count)
}
```

#### delete

在operations.go中新建函数Delete：

```go
func main() {
    defer Db.Close()
    //Insert()
    //Select()
    //Update()
    Delete()
}

func Delete()  {
    res, err := Db.Exec("delete from person where user_id = ?", 3)
    if err != nil {
       fmt.Println("exec failed, err:", err)
       return
    }
    count, err := res.RowsAffected()
    if err != nil {
       fmt.Println("exec failed, err:", err)
       return
    }

    fmt.Println("affected count:", count)
}
```

### MySQL事务

在operations.go中新建函数Transaction：

```go
func main() {
    defer Db.Close()
    //Insert()
    //Select()
    //Update()
    //Delete()
    Transaction()
}

func Transaction() {
    //开启事务
    conn, err := Db.Begin()
    if err != nil {
       fmt.Println("begin failed, ", err)
       return
    } else {
       fmt.Println("begin success")
    }
    //要执行的操作
    Insert()
    //提交事务或回滚事务
    err = conn.Commit()
    if err == nil {
       fmt.Println("commit success")
    }
    //conn.Rollback()
}
```

## Go操作Redis

### 准备工作

将第三方库下载到 GOPATH/pkg/ 路径中：

```bash
go get github.com/garyburd/redigo/redis
```

连接Redis：

新建redis-init.go：

```go
package main

import (
    "fmt"
    "github.com/garyburd/redigo/redis"
)

var Conn redis.Conn

func init() {
	conn2, err := redis.Dial("tcp", "hahhome:6379", redis.DialPassword("123456"))
	if err != nil {
		fmt.Println("conn redis failed,err:", err)
		return
	}
	Conn = conn2
}
```

### redis常用数据操作

执行命令都是使用 **redis.Do()** 来执行

#### set 和 get

新建operations.go：注意和redis-init.go在同一个包中共享一个init函数：

新建函数SetAndGet：

```go
func main() {
    defer Conn.Close()

    SetAndGet()
}

func SetAndGet() {
    _, err := Conn.Do("Set", "name", "小明")
    if err != nil {
       fmt.Println(err)
       return
    }

    reply, err := redis.String(Conn.Do("Get", "name"))
    if err != nil {
       fmt.Println(err)
       return
    }

    fmt.Println(reply)
}
```

#### String批量操作

operations.go中新建函数Strings：

```go
func main() {
    defer Conn.Close()

    Strings()
}

func Strings() {
    _, err := Conn.Do("mset", "name", "小明", "age", 20)
    if err != nil {
       fmt.Println(err)
       return
    }
    res, err := redis.Strings(Conn.Do("mget", "name", "age"))
    if err != nil {
       fmt.Println(err)
       return
    }
    for _, v := range res {
       fmt.Println(v)
    }
}
```

#### 设置过期时间

operations.go中新建函数Expire：

```go
func main() {
    defer Conn.Close()

    Expire()
}

func Expire() {
    _, err := Conn.Do("expire", "name", 60)
    if err != nil {
       fmt.Println(err)
       return
    }

    time.Sleep(time.Second * 2)

    reply, err := Conn.Do("ttl", "name")
    if err != nil {
       fmt.Println(err)
       return
    }
    fmt.Println(reply)
}
```

#### List队列操作

operations.go中新建函数ListOperations：

```go
func main() {
    defer Conn.Close()

    ListOperations()
}

func ListOperations() {
    _, err := Conn.Do("lpush", "book_list", "abc", "def", 300)
    if err != nil {
       fmt.Println(err)
       return
    }
    reply, err := redis.String(Conn.Do("lpop", "book_list"))
    if err != nil {
       fmt.Println(err)
       return
    }
    fmt.Println(reply)
}
```

#### Hash表操作

operations.go中新建函数HashOperations：

```go
func main() {
    defer Conn.Close()
    
    HashOperations()
}

func HashOperations() {
    _, err := Conn.Do("hset", "books", "price", 100)
    if err != nil {
       fmt.Println(err)
       return
    }
    reply, err := redis.Int(Conn.Do("hget", "books", "price"))
    if err != nil {
       fmt.Println(err)
       return
    }
    fmt.Println(reply)
}
```

### Redis连接池

```go
var pool *redis.Pool  //创建redis连接池

func init(){
	pool = &redis.Pool{     //实例化一个连接池
		MaxIdle:16,    //最初的连接数量
		// MaxActive:1000000,    //最大连接数量
		MaxActive:0,    //连接池最大连接数量,不确定可以用0（0表示自动定义），按需分配
		IdleTimeout:300,    //连接关闭时间 300秒 （300秒不使用自动关闭）
		Dial: func() (redis.Conn ,error){     //要连接的redis数据库
			return redis.Dial("tcp","hahhome:6379", redis.DialPassword("123456"))
		},
	}
}

func main() {
    defer pool.Close()//实际应用中是不关闭pool的

    conn := pool.Get()//从连接池，取一个连接
    defer conn.Close()

    _, err := conn.Do("set", "name", "小明")
    if err != nil {
       fmt.Println(err)
       return
    }

    reply, err := redis.String(conn.Do("get", "name"))
    if err != nil {
       fmt.Println(err)
       return
    }
    fmt.Println(reply)
}
```

## Go操作Kafka



## Go操作ElasticSearch