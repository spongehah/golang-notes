[TOC]

## slice

### slice 扩容

扩容源码：

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
	newcap = cap
} else {
	if old.len < 1024 {
		newcap = doublecap
	} else {
		// Check 0 < newcap to detect overflow
		// and prevent an infinite loop.
		for 0 < newcap && newcap < cap {
			newcap += newcap / 4
		}
		// Set newcap to the requested cap when
		// the newcap calculation overflowed.
		if newcap <= 0 {
			newcap = cap
		}
	}
}
```

从上面的代码可以看出以下内容：

- 首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）。
- 否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap），
- 否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）在满足条件下（0 < newcap && newcap < cap）从旧容量（old.cap）开始循环增加原来的1/4，直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
- 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）。

需要注意的是，切片扩容还会根据切片中元素的类型不同而做不同的处理，比如`int`和`string`类型的处理方式就不一样。



## for 和 range 的性能比较

> 参考链接：https://geektutu.com/post/hpg-range.html

总结：

- 遍历 []int, []*struct 的性能基本一致

- 遍历 []struct 的性能 range 远低于for

   - 并且：遍历 []struct 时，range 返回的是拷贝值，对返回的拷贝值进行修改不影响原来的切片。eg: 
      ```go
      persons := []struct{ no int }{{no: 1}, {no: 2}, {no: 3}}
      for _, s := range persons {
          s.no += 10
      }
      for i := 0; i < len(persons); i++ {
          persons[i].no += 100
      }
      fmt.Println(persons) // [{101} {102} {103}]
      ```

      























