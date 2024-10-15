## fmt

- 输入一行的内容并赋值给变量：

  ```go
  var msg string
  fmt.Scanln(&msg)
  ```

## strings

- 以某个字符串分割成数组：

  ```go
  strings.Split()
  ```

- 去掉字符串两端的某些字符：

  ```go
  strings.Trim()
  //去掉两端的空格：
  strings.TrimSpace()
  ```

- 将字符串拆分为单词数组：

  ```go
  strings.Fields()
  ```

## strconv

- 将 int 转换为 string

  ```go
  s := strconv.Itoa(i)
  ```

## sync

- 等待一组goroutine完成后执行后面的代码，起到阻塞同步的作用（类似于Java的CountDownLatch）：

  ```go
  var wg sync.WaitGroup
  wg.Add(1)//计数器加1，表示有一个 goroutine 需要等待完成
  go func() {
      defer wg.Done()//计数器减1,表示该 goroutine 已经完成
      ...
  }()
  wg.Wait()//阻塞下面代码的执行，直到计数器归零
  ...
  ```

- 确保某些操作在高并发的场景下只执行一次：

  ```go
  var loadOnce sync.Once
  loadOnce.Do(func())
  ```

## io

- socket通信时，监听对端的消息并打印到控制台的简写：

  ```go
  //一旦client.conn有数据，就直接copy到stdout的标准输出上，永久阻塞监听
  io.Copy(os.Stdout, this.conn)
  
  //等价于下面几句：
  //for {
  //  buf := make([]byte, 4096)
  //  n, err := this.conn.Read(buf)
  //  fmt.Println(string(buf[:n-1]))
  //}
  ```

## time

- 延时功能：

  ```go
  <-time.After(2*time.Second)//启动2秒的定时器
  ```

## flag

- 读取命令行参数：

  ```go
  var server string
  var port int
  
  func init() {
      //格式：flag.TypeVar("命令行参数", "要赋值的变量", default, usage)
      //命令行格式：./server -ip 127.0.0.1 -port 8888
      flag.StringVar("server", &server, "127.0.0.1", "请输入ip地址(默认是127.0.0.1)")
      flag.IntVar("ip", &port, "8888", "请输入端口号(默认是8888)")
  }
  ```

  