---
title: Golang调用C语言编写的动态库
abbrlink: 44670
date: 2022-11-18 13:33:33
tags: ["c语言", "golang", "so库", "动态链接库"]
cover: https://s2.loli.net/2022/11/18/KvsLg6YXrU154aj.jpg
categories: 
  - Golang
  - CGO
---

# 用C语言编写一个动态库
## 编写C语言头文件`sample_so.h`

在头文件中声明函数

```c
int export_func_add(int a, int b);
```

## 编写函数的实现代码`sample_so.c`

在源文件中实现函数

```c
#include "sample_so.h"

int export_func_add(int a, int b)
{
  return a + b;
}
```

# 编译生成动态链接库

```shell
gcc -c -fPIC -o sample_so.o sample_so.c # 先生成.o文件
gcc -shared -o libsample_so.so sample_o.o # 再生成.so文件
```

# golang 调用动态库

## 先用c语言编写一个动态库的` 调用代理`（名字瞎取的）

`loadso.h`

```c
int call_export_func(int a, int b);
```

`loadso.c`

```c
#include "loadso.h"
#include <dlfcn.h>

char dllname[] = "/path/of/the/so/sample_so.so";

int call_export_func(int a, int b) {
    void* handle;
    typedef int(*FPTR)(int, int);

    handle = dlopen(dllname, 1);
    if(handle == 0) {
        return -1;
    }

    FPTR fptr = (FPTR)dlsym(handle, "export_func_add");

    int result = (*fptr)(a, b);
    return result;
}
```

**注：实际上就是明确了动态库中的函数的签名后（就是函数参数类型，函数返回类型），然后定义对应的函数指针，根据函数名获取到这个函数，用函数指针指向它，接着就可以使用这个函数指针调用这个函数了**

## 编写golang代码

```go
package main

/*
#include "loadso.h"
#cgo LDFLAGS: -ldl
*/
import "C"
import "fmt"

func main() {
        var a int = 20
        var b int = 20
        fmt.Printf("%d + %d = %d\n", a, b, C.export_func_add(C.int(a), C.int(b)))
}
```

**注：看到最顶上的代码注释了吗？这个注释会被编译器进行解析，不是一般意义上的代码注释，个人感觉挺奇葩的。还有紧接着的一行`import "C"`, 注意要紧接着，这也挺反人类的**
