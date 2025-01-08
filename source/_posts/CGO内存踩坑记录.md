---
title: CGO内存踩坑记录
tags:
  - CGO
  - 内存
cover: 'https://s2.loli.net/2022/11/24/WLiMOjehw3zvR6H.png'
categories: 
  - Golang
  - CGO
abbrlink: 38363
date: 2022-11-24 10:56:03
---

# 应用场景

将 Go 内存中的字符串或字节切片传入 C 语言函数的应用场景。

# 里面的坑

假设一个极端场景：我们将一块位于某 `goroutine` 的栈上的` Go `语言内存传入了 `C `语言函数后，在此` C `语言函数执行时，如果 `goroutinue `的栈因为空间不足的原因发生了**扩展**，也就是导致了原来的 `Go `语言内存被移动到了新的位置。但是这时` C `语言函数并不知道该 `Go `语言内存已经移动了位置，仍然用之前的地址来操作该内存——也就是**“野指针”**。不是必现，要看内存中的数据是否发生移动。

以上操作本质上看是一种**“传引用”**，可以借助 C 语言内存稳定的特性（谁`malloc`就由谁来`free`，代码编写者自己对内存进行管理），在 C 语言空间先开辟同样大小的内存，然后将 Go 的内存填充到 C 的内存空间（**复制了一个副本过去**）。下面的例子是这种思路的具体实现：

# 例1

```go
package main

/*
#include <stdlib.h>
#include <stdio.h>

void printString(const char* s) {
    printf("%s", s);
}
*/
import "C"
import "unsafe"

func printString(s string) {
    cs := C.CString(s) // 这里会在C的内存空间里面开辟一块空间，把GO中的字符串s的值拷贝过去
    defer C.free(unsafe.Pointer(cs)) // 这里用defer 进行延迟操作，释放这块在C里面开辟的空间，不然就是内存泄漏

    C.printString(cs)
}

func main() {
    s := "hello"
    printString(s)
}
```

这种方式虽然解决了问题，但是，写法有点啰嗦，还要自己手动`C.free`。最主要的是：性能效率有损失，因为要从`GO`环境把内存复制到`C`语言环境内存中。

# 例2

```go
package main

/*
#include<stdio.h>

void printString(const char* s, int n) {
    int i;
    for(i = 0; i < n; i++) {
        putchar(s[i]);
    }
    putchar('\n');
}
*/
import "C"

func printString(s string) {
    p := (*reflect.StringHeader)(unsafe.Pointer(&s)) // 把字符串的内存地址赋值给了p
    C.printString((*C.char)(unsafe.Pointer(p.Data)), C.int(len(s))) // 传入参数
}

func main() {
    s := "hello"
    printString(s)
}
```

现在的处理方式更加直接，且避免了分配额外的内存，而且CGO会保证在C函数调用期间（从C语言函数被调用到C语言函数调用结束），传入的指针指向的这块GO语言开辟的内存不会被移动。注意：~~**`CGO只会保证在C函数调用期间`**。~~ 在网上有看到这个说法，但是我实践中似乎有出入。我实践中就是用这个用法，结果GG了，我实践中它好像也不能保证这一点，这个后续再研究研究。

就算CGO会保证在C函数调用期间保证内存不会被移动，这里还有有一个容易踩的坑：

```go
// 错误的代码
p := (*reflect.StringHeader)(unsafe.Pointer(&s)) // 把字符串的内存地址赋值给了p
tmp := uintptr(p)
ptr := (*reflect.StringHeader)(unsafe.Pointer(tmp))
C.printString((*C.char)(unsafe.Pointer(ptr.Data)), C.int(len(s))) // 传入参数
```

因为 tmp 并不是指针类型(它只是表示内存地址，但并没有指针的**语义**，指针和内存地址还是有点差别的，差别在于指针是有数据类型的，是和指向的内存中的数据进行了“绑定的”，而内存地址仅仅是地址而已)。在它获取到` Go `对象地址之后`s`对象可能会被移动，但是因为不是`tmp`没有指针语义，所以不会被` Go` 语言运行时更新成新内存的地址(`s`对象新的移动后的位置）。在`tmp`中保持 `Go` 对象的地址，和在` C` 语言环境保持` Go` 对象的地址的效果是一样的：如果原始的` Go `对象内存发生了移动，Go 语言运行时并不会同步更新它们。这样就还是会触发一开始的那个问题。

所以，还是慎用例2的方法吧

**续：**我去看了`StringHeader`的数据结构和`SliceHeader`数据结构

```go
// StringHeader is the runtime representation of a string.
// It cannot be used safely or portably and its representation may
// change in a later release.
// Moreover, the Data field is not sufficient to guarantee the data
// it references will not be garbage collected, so programs must keep
// a separate, correctly typed pointer to the underlying data.
type StringHeader struct {
	Data uintptr
	Len  int
}

// SliceHeader is the runtime representation of a slice.
// It cannot be used safely or portably and its representation may
// change in a later release.
// Moreover, the Data field is not sufficient to guarantee the data
// it references will not be garbage collected, so programs must keep
// a separate, correctly typed pointer to the underlying data.
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

看到`Data`的类型是`uintptr`就感觉不妙了...这里自己实现一个可能能解决问题，我能想到的，go标准库的开发者会想不吗？肯定是有什么原因的。所以，还是最好不用`StringHeader`了。

```go
type StringHeader struct {
	Data unsafe.Pointer
	Len  int
}
```

