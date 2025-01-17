> 摘抄/总结自 https://geektutu.com/post/high-performance-go.html

## benchmark基准测试

示例代码：

```go
package main

import "testing"

func fib(n int) int {
	if n == 0 || n == 1 {
		return n
	}
	return fib(n-2) + fib(n-1)
}

func BenchmarkFib(b *testing.B) {
	for n := 0; n < b.N; n++ {
		fib(30) // run fib(30) b.N times
	}
}
```



运行示例：

```sh
$ go test -bench='Generate' -benchmem .
goos: darwin
goarch: amd64
pkg: example
BenchmarkGenerateWithCap-8  43  24335658 ns/op  8003641 B/op    1 allocs/op
BenchmarkGenerate-8         33  30403687 ns/op  45188395 B/op  40 allocs/op
PASS
ok      example 2.121s

# 43代表b.N=43，运行了43次，
# 24335658 ns/op表示没次运行的平均耗时，
# 8003641 B/op表示没次运行分配的内存大小，
# 1 allocs/op表示没次运行分配内存的次数，
# benchmark默认运行时间是1s，b.N为1s运行尽可能多的次数
```



- 运行：go test -bench="pattern" .
- 可带参数：
   - -benchmem：展示内存分配情况
   - -benchtime=5s：执行5s
   - -benchtime=50x：运行50次
   - -count=3：进行3轮benchmark
   - -cpu=2,4：设置CPU 核数
- 可使用testing.B的方法对耗时进行操作：
   - b.ResetTimer
   - b.StopTimer
   - b.StartTimer



## pprof性能分析



















