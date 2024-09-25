# Golang设计模式

> Reference Links
>
> - https://www.liwenzhou.com/posts/Go/functional-options-pattern/

## 函数选项模式

函数选项模式（Functional Options Pattern）也称为选项模式（Options Pattern），是一种创造性的设计模式，允许你使用接受**零个**或**多个**函数作为参数的可变构造函数构建复杂结构。我们将这些函数称为选项，由此得名函数选项模式。

对于字段过多结构体，怎么设置默认值？怎么可选的设置参数？
作用：避免编写**字段过多**的结构体声明函数时，非常不优雅

### struct 版本

```go
type Options struct {
    a string
	b int
	c bool
	// ...
}

// 传入一个 *Options，对它的字段值进行修改
type OptionFunc func(*Options)

// With 函数返回OptionFunc，在NewOptions中调用With函数然后返回一个OptionFunc
func WithA(a string) OptionFunc {
    return func(o *Options) {
        o.a = a
    }
}

// WithB, WithC ...

// 真正的new函数
func NewOptions(opts ...OptionFunc) *Options {
    // 先设置默认值
    o := &Options{
        a: defaultA,
        b: defaultB,
        c: defaultC,
    }
    // 便利执行所有的With函数
    for _, opt := range opts {
        opt(o)
    }
    return o
}
```



### interface 版本

在一些场景下，我们可能并不想对外暴露具体的配置结构体，而是仅仅对外提供一个功能函数。这时我们会将对应的结构体定义为小写字母开头，将其限制只在包内部使用。

```go
package main

// 私有的，不对外暴露的
type options struct {
	a string
	b int
	c bool
	// ...
}

// IOption 其中 apply 函数的作用，在 struct 版本中的角色相当于 OptionFunc
type IOption interface {
	apply(*options)
}

// funcOption 实现了 IOption 接口
type funcOption struct {
	f func(*options) // 实现的 apply 函数，在 struct 版本中的角色相当于 OptionFunc
}

// apply 实现 IOption 接口的方法
func (fo *funcOption) apply(o *options) {
	fo.f(o)
}

func newFuncOption(f func(*options)) IOption {
	return &funcOption{
		f: f,
	}
}

func WithA(a string) IOption {
	return newFuncOption(func(o *options) {
		o.a = a
	})
}

// WithB, WithC ...

// DoSomething 暴露给外部的函数
func DoSomething(opts ...IOption) {
	// 先设置默认值
	o := &Options{
		a: defaultA,
		b: defaultB,
		c: defaultC,
	}
	for _, opt := range opts {
		opt.apply(o)
	}
	// doSomething
}
```

































