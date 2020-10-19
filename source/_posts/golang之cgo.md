---
title: golang 之 cgo
date: 2018-08-30 18:00:12
index_img:
- /images/golang/string.jpg
tags: 
- golang cgo
categories:
- golang
---

使用golang开发有段时间，公司内部很多产品都是用golang开发的，除了底层核心的算法层是用C++ 写的，其他基本都是利用go进行封装对外提供给客户。由于底层是使用的是c++，上层使用的go，所以避免不了使用的cgo了。那什么是cgo呢？简单来说cgo支持创建调用C代码的Go包。下面我们来一步一步介绍cgo。

### cgo 的简单例子

让我们从一个简单的cgo 例子说起

```go
package main
//#include <stdio.h>
//#include <stdlib.h>
import "C" //该语句要单独写一行

import (
   "unsafe"
)

func main() {
   //C.CString将go string 转换成C string
   s := C.CString("Hello, World\n")
   // C.CString() 返回的 C 字符串是在堆上新创建的并且不受 GC 的管理，
   // 使用完后需要自行调用 C.free() 释放，否则会造成内存泄露，
   // 而且这种内存泄露pprof 也定位不出来
   defer C.free(unsafe.Pointer(s))
   //C.puts 函数向标准输出窗口打印，
   C.puts(s)
}
```

虽然上面的代码有注释了，但是还是有一些需要特别注意的。

1. 第二行到第五行，需要注意的是#include 前面的//，不要认为是注释。这里的意思是引用了C的标准库。这个//的注释是叫preamble。必须和底下的import “C” 中间挨着不能有空行的。否则会导致后面的编译无法通过。当然除了//也可以使用/**/。同时这里的import “C” 需要和其他的import 隔开，并不能放到同一个import中，否则程序无法检测是否包含cgo

   ```go
   //#include <stdio.h>
   //#include <stdlib.h>
   import "C"
   ```

2. 第二个是main 函数里面的内容

   ```go
   func main() {
      //C.CString将go string 转换成C string
      s := C.CString("Hello, World\n")
      // C.CString() 返回的 C 字符串是在堆上新创建的并且不受 GC 的管理，
      // 使用完后需要自行调用 C.free() 释放，否则会造成内存泄露，
      // 而且这种内存泄露pprof 也定位不出来
      defer C.free(unsafe.Pointer(s))
      //C.puts 函数向标准输出窗口打印，
      C.puts(s)
   }
   ```

   - Go代码中的`s`变量在传递给c代码使用完成之后，需要调用`C.free`进行释放。首先我们需要先看下golang的字符串和C中的字符串在底层中的内存模型。

     ![](/images/golang/string.jpg)

     从上图中我们可以看出golang 和 C 字符串在底层中的内存模型是不一样的。golang 字串符串并没有用 ‘\0’ 终止符标识字符串的结束，因此直接将 golang 字符串底层数据指针传递给 C 函数是不行的。一种方案类似切片的传递一样将字符串数据指针和长度传递给 C 函数后，C 函数实现中自行申请一段内存拷贝字符串数据然后加上未层终止符后再使用。更好的方案是使用标准库提供的 C.CString()将 golang 的字符串转换成 C 字符串然后传递给 C 函数调用。

     我们看下官方文档对CString的说明。所以我们也就知道C.free的作用是什么了

     ```go
     // Go string to C string
     // The C string is allocated in the C heap using malloc.
     // It is the caller's responsibility to arrange for it to be
     // freed, such as by calling C.free (be sure to include stdlib.h
     // if C.free is needed).
     func C.CString(string) *C.char
     
     // 这个文档上的大概意思是
     // C string在C的堆上使用malloc申请。调用者有责任在合适的时候对该字符串进行释放，释放方式可以是调用C.free（调用C.free需包含stdlib.h）
     ```

     我们来看看其他几种C类型和Go类型相互转换的。可以看出CBytes 也是需要我们手动free的。

     ```go
     // Go []byte slice to C array
     // The C array is allocated in the C heap using malloc.
     // It is the caller's responsibility to arrange for it to be
     // freed, such as by calling C.free (be sure to include stdlib.h
     // if C.free is needed).
     func C.CBytes([]byte) unsafe.Pointer
     
     // C string to Go string
     func C.GoString(*C.char) string
     
     // C data with explicit length to Go string
     func C.GoStringN(*C.char, C.int) string
     
     // C data with explicit length to Go []byte
     func C.GoBytes(unsafe.Pointer, C.int) []byte
     ```

     一般来说，c语言里都是秉承哪个模块申请就由哪个模块释放的原则，因为跨库申请释放可能由于各自链接的内存管理库不一致导致出现难以排查的bug。并且换个角度来说，被调用模块也无法知道传入的内存是在堆上申请还是栈上申请的，是否需要释放。
     所以Go传入c模块的内存，c模块也许会对这块内存再次进行拷贝，但是c模块肯定不会释放（即`free`）传入的这份内存。
     所以，一般来说，Go在调用完c函数之后，Go需要释放拷贝生成的这块内存。

     

   - 其中需要留意的是unsafe.Pointer 这个函数。消除内存拷贝可以通过`unsafe.Pointer`获取原始指针进行传递。`unsafe.Pointer`是一种特殊意义的指针，它可以包含任意类型的地址，有点类似于C语言里的void*指针，全能型的。

Go语言中数值类型和C语言数据类型基本上是相似的。但是还是有不同的。这里就不详细介绍了。更多的cgo 类型装换可以参考[Go语言高级编程](https://books.studygolang.com/advanced-go-programming-book/ch2-cgo/ch2-03-cgo-types.html)

### Go 和 C 之间的数组传递

#### Go数组到C 

我们还是直接来看下例子。我们在C 中遍历数组，并计算数组内值的和。

```go
package main
/*
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int loop(int** list_data, int leng, char** data)
{
  int* m = (int*)list_data;
  int sum = 0;
  for(int i=0; i<leng; i++)
  {
    sum += m[i];
  }
  *data = "finised task";
  return sum;
}
*/
import "C" //该语句要单独写一行
import (
	"unsafe"
	"fmt"
)
func GoSilence2CArray() {
	var ids = []int32{1, 2, 3, 5}
	var res *C.char
	length := C.int(len(ids))
	le := C.loop((**C.int)(unsafe.Pointer(&ids[0])), length, &res)
	fmt.Println(le)
	fmt.Println(C.GoString(res))
	fmt.Println(ids)
}
func main() {
	GoSilence2CArray()
}
```

整个例子需要注意的就是调用C.loop 函数的时候，将Go中的Slice传给C函数。其中最重要的loop的第一个参数，将数组转换的代码了。我们来看以下使用loop函数的写法。

```go
//正确写法
le := C.loop((**C.int)(unsafe.Pointer(&ids[0])), length, &res)
//错误写法
le := C.loop((**C.int)(unsafe.Pointer(&ids)), length, &res)
```

Slice在Go中实际上不是一个完全意义上的数组，它只是一种数据结构，带有若干头部。如果直接&ids，那相当于把ids这个数据结构的地址处的数据强制转换为`(**C.int)`。这样导致的后果完全不可期，运行时core掉是再正常不过的后果。所以正确的写法应该是`(**C.int)(unsafe.Pointer(&ids[0]))`。从slice的第一个元素地址开始。对于slice的内部数据结构可以查看[slice的使用和内部结构](http://blog.golang.org/go-slices-usage-and-internals)

#### C数组到Go slice

还是直接看例子

```go
package main
/*
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct{
   char* name;
}person;

person* get_person(int n){
   person* ret = (person*)malloc(sizeof(person) * n);
   for(int i=0;i<n;i++){
      ret[i].name="wu";
   }
   return ret;
}
*/
import "C" //该语句要单独写一行
import (
	"unsafe"
	"fmt"
)
func CArray2GoSilence() {
	size := 2
	person := C.get_person(C.int(size))
	person_array := (*[1 << 30]C.person)(unsafe.Pointer(person))
	var names []string
	for i := 0; i < size; i++ {
		name := C.GoString(person_array[i].name)
		names = append(names, name)
	}
	for _, name := range names {
		fmt.Println(name)
	}
	C.free(unsafe.Pointer(person))
}
func main() {
	CArray2GoSilence()
}
```

C语言中的数组与Go语言中的数组差异较大，后者是值类型，而前者与C中的指针大部分场合都可以随意转换。目前似乎无法直接显式的在两者之间进行转型，官方文档也没有说明。但我们可以通过编写转换函数，将C的数组转换为Go的Slice。`(*[1 << 30]C.person)(unsafe.Pointer(person))` 是获取足够大的内存地址，然后通过遍历该数组的地址。获取数组的元素。可以在for中将元素添加到slice中。

### cgo利用pkg_config 使用第三方的so库

#### pkg_config

我们在使用第三方库的时候就少不了使用第三方库的头文件和库文件。我们在编译、链接的时候，必须要指定这些头文件和库的位置。**pkg-config** 是一个在[源代码](https://zh.wikipedia.org/wiki/源代码)[编译](https://zh.wikipedia.org/wiki/编译)时查询已安装的[库](https://zh.wikipedia.org/wiki/库)的使用接口的计算机工具[软件](https://zh.wikipedia.org/wiki/软件)。pkg-config原本是设计用于[Linux](https://zh.wikipedia.org/wiki/Linux)的，但现在在各个版本的[BSD](https://zh.wikipedia.org/wiki/BSD)、[windows](https://zh.wikipedia.org/wiki/Windows)、[Mac OS X](https://zh.wikipedia.org/wiki/Mac_OS_X)和[Solaris](https://zh.wikipedia.org/wiki/Solaris)上都有着可用的版本。

那么pkg-config有什么作用呢？

1. 检查库的版本号。如果所需要的库的版本不满足要求，它会打印出错误信息，避免链接错误版本的库文件。
2. 获得编译预处理参数，如宏定义，头文件的位置。
3. 得链接参数，如库及依赖的其它库的位置，文件名及其它一些连接参数。
4. 自动加入所依赖的其它库的设置。

pkg-config是如和得到这些信息呢？为了让pkg-config可以得到这些信息，要求库的提供者，提供一个.pc文件。以下是这是一个用于[libpng](https://zh.wikipedia.org/w/index.php?title=Libpng&action=edit&redlink=1)的.pc文件的样例:

```shell
prefix=/usr/local
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${exec_prefix}/include
  
Name: libpng12
Description: Loads and saves PNG files
Version: 1.2.8
Libs: -L${libdir} -lpng12 -lz
Cflags: -I${includedir}/libpng12
```

这个文件告诉我们这些库可以在/usr/local/lib找到，头文件可以在/usr/local/include里找到，库的名字是libpng12并且版本号是1.2.8。它也提供了用于编译依赖于libpng的源代码时需要的链接器参数。

pkg-config默认会去/usr/local/lib/pkgconfig目录下，寻找.pc文件。

#### cgo 使用pkg-config

我们先来看下cgo 中使用pkg-config的例子。以下是我做餐盘识别项目中涉及到cgo部分

```go
package cgo
// #cgo pkg-config: megproduct
// #cgo CXXFLAGS: -std=c++11
// #include "plate.h"
// #include "stdlib.h"
import "C"
//后续使用方法和go 调用C的方式一致了
```

我们可以看到cgo有配置pkg-config的配置文件megproduct.pc，这个pc文件可以放在/usr/local/lib/pkgconfig目录下，也可以将其添加到环境变量PKG_CONFIG_PATH中。根据megproduct.pc 查找库的头文件和so文件。

我们来看下megproduct.pc 是咋样的

```shell
prefix=/go/DATA/plate-sdk-190710
libdir=${prefix}/lib
includedir=${prefix}/include

Name: megproduct
Description: megproduct, built in megproduct
Version: 1.1
Requires:
Libs: ${libdir}/libmegproduct.so
Libs.private: -Wl,--push-state,--as-needed,-lpthread,-lrt,-ldl,-lm,--pop-state
Cflags: -I${includedir}
```

使用pkg-config少了很多配置的文件。所以如果使用第三方库进行cgo 开发还是建议使用pkg-config



本篇文章更多讲的是cgo的使用，并没有涉及到底层cgo的内部实现，后续如果有时间会对cgo 底层的实现讲解。



参考：[Go语言使用cgo时的内存管理笔记](https://www.pengrl.com/p/29054/)

​			[官方文档](https://golang.org/cmd/cgo/)

​			[slice的使用和内部结构](http://blog.golang.org/go-slices-usage-and-internals)