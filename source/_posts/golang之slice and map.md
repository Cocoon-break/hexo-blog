---
title: golang 之 slice & defer 关键字
date: 2018-09-10 19:53:22
index_img:
- /images/golang/logo.jpg
tags: 
- golang
categories:
- golang
---

今天我们讲讲golang的slice和defer。

slice是go里面一个很重要的数据结构，使用这种结构来管理数据集合。slice 类似其他语言中的数组，但是又有一些其他不同的属性。

在正式开始讲slice之前我们来说一说golang 中的`defer`，原本觉得自己对defer有一定了解了。但是昨天看了一本书，才发现自己对defer 的了解是这么浅，在没有运行以下代码前你能否得出正确答案呢？运行代码之后能否回答为什么？

```go
package main
import "fmt"
func f() (result int) {
	defer func() {
		result++
	}()
	return 0
}
func f2() (r int) {
	t := 5
	defer func() {
		t = t + 5
	}()
	return t
}
func f3() (r int) {
	defer func(r int) {
		r = r + 5
	}(r)
	return 1
}

func main() {
	fmt.Println(f())
	fmt.Println(f2())
	fmt.Println(f3())
}
```

答案留在最后讲解

### slice

#### 数组

在说slice之前，我们先来了解下数组。

数组是指定长度和元素类型的数据集合。比如以下数组

```go
var intArr [3]int//创建了长度为3的int数组
a[0]=1 //根据索引访问元素
fmt.println(a[1])//int 类型数组默认值为0
```

数组不需要显式初始化。Go的数组是值类型。数组变量表示整个数组，它不是指向数组的第一个元素的指针，这就意味着赋值和函数传参操作时，将会复制数组的内容（后续和slice对比，slice是创建指针指向地址）。可以写一个简单的程序将数组的地址打印出来，进行对比。

#### slice

在实际使用中我们很少使用到数组，因为数组不够灵活。比如：一旦数组中的数据足够大，每次使用数组都要重新复制一遍，耗费大量内存和时间。slice并不会实际复制一份数据，它只是创建一个新的数据结构，包含了另外的一个指针，一个长度和一个容量数据。我们来看下slice在源码中的定义

```go
type slice struct {
	array unsafe.Pointer// 指向底层数组的指针
	len   int// 长度，切片可用元素的个数，slice的下标不能超过长度 
	cap   int// 容量 >= 长度，在底层不扩容的情况下，cap是len的最大限度
}
```

需要注意的是：底层数组是可以被多个slice同时指向的，也就是对一个slice的元素进行修改会影响到其他的slice。

##### slice的扩容

在对slice进行append等操作时，可能会造成slice的自动扩容。我们来看下源码是怎么扩容的

```go
// growslice handles slice growth during append.
// It is passed the slice element type, the old slice, and the desired new minimum capacity,
// and it returns a new slice with at least that capacity, with the old data
// copied into it.
// The new slice's length is set to the old slice's length,
// NOT to the new requested capacity.
// This is for codegen convenience. The old slice's length is used immediately
// to calculate where to write new values during an append.
// TODO: When the old backend is gone, reconsider this decision.
// The SSA backend might prefer the new length or to return only ptr/cap and save stack space.
func growslice(et *_type, old slice, cap int) slice {
	//忽略部分源码
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
//忽略部分源码
	return slice{p, old.len, newcap}
}
```

当slice中的cap不够使用是会调用growslice函数进行扩容，具体的扩容规则是

- 如果新的大小是当前大小2倍以上，则大小增长为新大小
- 如果当前大小小于1024，按每次2倍增长，否则每次按当前大小1/4增长。直到增长的大小超过或等于新大小。

扩容之后，向 Go 内存管理器申请内存，将老 slice 中的数据复制过去，并且将 append 的元素添加到新的底层数组中。最后向 `growslice` 函数调用者返回一个新的 slice，这个 slice 的长度并没有变化，而容量却增大了。

##### nil slice 和空slice

nil slice 是描述一个不存在的切片，也就是说它的指针指向nil，没有实际上地址，长度和容量也都为0。

而空的切片是是描述空的一个集合，但是它的指针指向了一个地址，长度和容量也为0。

nil切片和空切片很相似，长度和容量都是0，官方建议尽量使用 `nil` 切片。

| 创建方式  | nil切片              | 空切片                  |
| --------- | -------------------- | ----------------------- |
| 方式一    | var s1 []int         | var s2 = []int{}        |
| 方式二    | var s4 = *new([]int) | var s3 = make([]int, 0) |
| 长度      | 0                    | 0                       |
| 容量      | 0                    | 0                       |
| 和nil比较 | true                 | false                   |

#### slice 和unsafe.Pointer相互转换

在上一篇的cgo 部分中将c的数组转到go slice 部分中有涉及到unsafe.Pointer。

我们先来看看从slice中获取一块内存地址

```go
slice:=make([]int,10)
ptr:=unsafe.Pointer(&slice[0])//获取数组中第一个元素的内存地址
```

从内存指针构造出Go的slice结构会比较麻烦，总共有三种方式

第一种，先将`ptr`强制类型转换为另一种指针，一个指向`[1<<10]int`数组的指针，这里数组大小其实是假的。然后用slice操作取出这个数组的前10个，于是`s`就是一个10个元素的slice。

```go
var ptr unsafe.Pointer
s := ((*[1<<10]int)(ptr))[:10]
```

第二种，模拟go 底层的slice结构，将结构体赋值给s。和第一种相比，这里的cap和1<<10意思是相同的

```go
var ptr unsafe.Pointer
var s1 = struct {
    addr uintptr
    len int
    cap int
}{ptr, length, length}
s := *(*[]byte)(unsafe.Pointer(&s1))
```

第三种方法，通过reflect.SliceHeader的方式来构造slice。

```go
var o []byte
sliceHeader := (*reflect.SliceHeader)((unsafe.Pointer(&o)))
sliceHeader.Cap = length
sliceHeader.Len = length
sliceHeader.Data = uintptr(ptr)
```

### defer

不知道你是否得出文章开始代码的正确结果呢？

 `defer` 会在当前函数或者方法返回之前执行传入的函数。它会经常被用于关闭文件描述符、关闭数据库连接以及解锁资源。要真正理解文章前面的问题，我们要理解**return xxx这一条语句并不是一条原子指令!**

函数返回的过程是这样的：先给返回值赋值，然后调用defer表达式，最后才是返回到调用函数中。

defer表达式可能会在设置函数返回值之后，在返回到调用函数之前，修改返回值，使最终的函数返回值与你想象的不一致。

其实使用defer时，用一个简单的转换规则改写一下，就不会迷糊了。改写规则是将return语句拆成两句写，return xxx会被改写成:

```shell
返回值 = xxx
调用defer函数
空的return
```

 这样你就能理解为啥会得出文章一开始函数的结果了。我们来查看下改写之后的代码

```go
func f() (result int) {
	result = 0 //return语句不是一条原子调用，return xxx其实是赋值＋ret指令
	func() { //defer被插入到return之前执行，也就是赋返回值和ret指令之间
		result++
	}()
	return
}
func f2() (r int) {
	t := 5
	r = t //赋值指令
	func() { //defer被插入到赋值与返回之间执行，这个例子中返回值r没被修改过
		t = t + 5
	}()
	return //空的return指令
}
func f3() (r int) {
	r = 1 //给返回值赋值
	func(r int) { //这里改的r是传值传进去的r，不会改变要返回的那个r值
		r = r + 5
	}(r)
	return //空的return
}
```

接下来我们来看看另外一小段代码

```go
func main(){
  startedAt := time.Now()
	defer fmt.Println(time.Since(startedAt))
	time.Sleep(time.Second)
}
//输出结果并不是1s，而是几百纳秒
```

上面的输出结果背后的原因是什么呢？经过分析，我们会发现调用 `defer` 关键字会立刻对函数中引用的外部参数进行拷贝，所以 `time.Since(startedAt)` 的结果不是在 `main` 函数退出之前计算的，而是在 `defer` 关键字调用时计算的，最终导致上述代码输出。当然上面的解决办法是很简单的。只要defer 那添加一个匿名函数就可以解决了。我们总结一下`defer` 关键字使用传值的方式传递参数时会进行预计算，导致不符合预期的结果。



参考：[深入解析go中slice底层实现](https://halfrost.com/go_slice/)

[深度解析Go语音之slice](https://www.cnblogs.com/qcrao-2018/p/10631989.html)

[深入解析Go](https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.4.html)

