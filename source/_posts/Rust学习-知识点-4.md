---
title: Rust学习-知识点-4
tags:
  - Result
  - Option
  - 错误传播
  - 错误处理
cover: 'https://s2.loli.net/2022/10/21/zENYfxD3SXjhwbW.jpg'
categories: 
  - Rust
  - 基础
abbrlink: 15838
date: 2022-10-19 14:22:44
---

# 传播错误

程序中涉及到的函数调用往往会嵌套多层，而错误的处理也往往不是在哪出错，就在哪里处理，底层被调用的函数一般会把错误上传交给调用者去处理。因为上层调用方一般会涉及业务逻辑，底层不对错误做相应的业务处理。比如：

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    // 打开文件，f是`Result<文件句柄,io::Error>`
    let f = File::open("hello.txt");

    let mut f = match f {
        // 打开文件成功，将file句柄赋值给f
        Ok(file) => file,
        // 打开文件失败，将错误返回(向上传播)
        Err(e) => return Err(e),
    };
    // 创建动态字符串s
    let mut s = String::new();
    // 从f文件句柄读取数据并写入s中
    match f.read_to_string(&mut s) {
        // 读取成功，返回Ok封装的字符串
        Ok(_) => Ok(s),
        // 将错误向上传播
        Err(e) => Err(e),
    }
}
```

`read_username_from_file()`内部一般只需要向上抛出错误即可；该函数的调用者最终会对错误进行处理，至于怎么处理就是调用者的事，可以选择向上传播，也可以直接`panic`，亦或将具体的错误原因包装后呈现给用户。

虽然错误是传播出去了，但是，代码写起来有点长，`rust` 提供了`?`来简化错误的传播。

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

代码量减少明显，其实`?`就是一个宏，它的作用中上面的`match`一样：

```rust
let mut f = match f {
    // 打开文件成功，将file句柄赋值给f
    Ok(file) => file,
    // 打开文件失败，将错误返回(向上传播)
    Err(e) => return Err(e),
};
```

但`?`还有一点比`match`更强的地方，想像一下，一个系统中，肯定有自定义的错误特征，错误之间很可能会存在上下级关系，例如标准库中的`std::io:Error`和`std::error:Error`，前者是IO相关的错误结构体，后者是一个最通用的标准错误特征，同时前者实现在了后者，因此`std::io::Error`可以转换为`std::error::Error`。

明白了上述的错误转换，再来看下面的一段代码来体会一下`?`比`match`更强的地方：

```rust
fn open_file() -> Result<File, Box<dyn std::error::Error>> {
    let mut f = File::open("hello.txt")?;
    Ok(f)
}
```

`?`还可以实现链式调用，写起代码来更加连贯：

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

# `Option`的传播

`?`不仅仅可以用于`Result`的传播，还能用于`Option`的传播，`Option`的定义如下：

```rust
pub enum Option<T> {
  Some(T),
  None,
}
```

`Result`通过`？`反回错误，那么`Option`就通过`?`返回None：

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   let v = arr.get(0)?;
   Some(v)
}
```

上面的代码只是为了演示`Option`的传播，从功能上来说，完全可以用以下简化的版本：

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)
}
```

当然，开发中肯定有`Option`传播的应用场景的：

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

上面代码展示了在链式调用中使用`?`提前返回`None`的用法，`.next`方法返回的是`Option`类型：如果返回`Some(&str)`，那么继续调用`chars`方法，如果返回`None`，则直接`last_char_of_first_line`返回,返回`None`，不再继续链式调用。

