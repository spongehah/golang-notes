# Golang标准库

> Reference Links:
>
> - https://www.liwenzhou.com/posts/Go/context/

[TOC]

## context

### 作用和意义

Go1.7加入了一个新的标准库`context`，它定义了`Context`类型，专门用来简化 对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个 API 调用。

对服务器传入的请求应该创建上下文，而对服务器的传出调用应该接受上下文。它们之间的函数调用链必须传递上下文，或者可以使用`WithCancel`、`WithDeadline`、`WithTimeout`或`WithValue`创建的派生上下文。当一个上下文被取消时，它派生的所有上下文也被取消。

当一个请求被取消或超时时，所有用来处理该请求的 goroutine 都应该迅速退出，然后系统才能释放这些 goroutine 占用的资源。



### context.Context 接口

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- `Deadline`方法需要返回当前`Context`被取消的时间，也就是完成工作的截止时间（deadline）；
- `Done`方法需要返回一个`Channel`，这个Channel会在当前工作完成或者上下文被取消之后关闭，多次调用`Done`方法会返回同一个Channel；
- `Err`方法会返回当前`Context`结束的原因，它只会在`Done`返回的Channel被关闭时才会返回非空的值；
   - 如果当前`Context`被取消就会返回`Canceled`错误；
   - 如果当前`Context`超时就会返回`DeadlineExceeded`错误；
- `Value`方法会从`Context`中返回键对应的值，对于同一个上下文来说，多次调用`Value` 并传入相同的`Key`会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域的数据；



### Background () 和TODO()

Go内置两个函数：`Background()`和`TODO()`，这两个函数分别返回一个实现了`Context`接口的`background`和`todo`。我们代码中最开始都是以这两个内置的上下文对象作为最顶层的`partent context`，衍生出更多的子上下文对象。

`Background()`主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context。

`TODO()`，它目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个。

`background`和`todo`**本质上都是`emptyCtx`结构体类型**，是一个不可取消，没有设置截止时间，没有携带任何值的Context。



### With创建context

1. **WithCancel()**

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

功能：

- 创建一个新的上下文，该上下文继承自父上下文。并且提供一个 `cancel` 函数，可以用来手动取消这个上下文。

上下文的取消条件：
新创建的上下文的 `Done()` 通道会在以下情况中最先发生的一种情况下关闭：

- 调用返回的 `cancel` 函数
- 父上下文的 `Done()` 通道关闭

2. **WithDeadline()**

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
```

功能：

- 创建一个新的上下文，该上下文继承自父上下文，但增加了一个不晚于 `deadline` 的截止时间。
- 如果父上下文已经有一个更早的截止时间，新上下文将使用父上下文的截止时间。

上下文的取消条件：
新创建的上下文的 `Done()` 通道会在以下情况中最先发生的一种情况下关闭：

- 达到设定的截止时间
- 调用返回的取消函数
- 父上下文的 `Done()` 通道关闭

3. **WithTimeout()**

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

实际上就是`WithDeadline(parent, time.Now().Add(timeout))`

功能和上下文取消条件与 `WithDeadline`  一致

4. **WithValue()**

```go
func WithValue(parent Context, key, val interface{}) Context
```

功能：

- 创建一个新的上下文，该上下文继承自父上下文，并添加了一个键值对。
- 允许在上下文中存储和传递请求作用域的数据。

使用方法：

- 存储值：使用 `WithValue` 创建新的上下文时添加键值对。
   - 键必须是**可比较**的；
   - 键不应该是 string 类型或任何其他**内置类型**;
- 获取值：在后续的函数调用中使用 `ctx.Value(key)` 来获取存储的值。

```go
type TraceCode string

func main() {
	// TraceCode 是自定义类型
	ctx = context.WithValue(ctx, TraceCode("TRACE_CODE"), "123456")
}
```



### 注意事项

- 推荐以参数的方式显示传递Context
- 以Context作为参数的函数方法，应该把Context作为第一个参数。
- 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO()
- Context的Value相关方法应该传递请求域的必要数据，不应该用于传递可选参数
- Context是线程安全的，可以放心的在多个goroutine中传递





























