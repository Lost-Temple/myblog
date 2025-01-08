---
title: Rust学习-知识点-3
tags:
  - Rc<T>
  - Box<T>
  - 链表
  - 嵌套
categories: 
  - Rust
  - 基础
cover: 'https://s2.loli.net/2022/10/14/JAjkKl3hcapqr86.webp'
abbrlink: 65439
date: 2022-10-14 14:22:44
---

# 拥有共享数据的链表

以一道题开始吧，尝试用`Box<T>`实现下面的链表

![链表.webp](https://s2.loli.net/2022/10/14/JAjkKl3hcapqr86.webp)

# 使用`Box<T>`实现

```rust
use crate::List::{Cons, Nil};

enum List {
    Cons(i32, Box<List>),
    Nil,
}

fn main() {
    // 创建链表a
    let a = Cons(5, Box::new(Cons(10, Box::new(List::new(Nil)))));

    // 创建链表b， 并链接到链表a
    let b = Cons(3, Box::new(a)); // b链接到a时，就把a的所有权给转移走了

    // 创建链表c，并连接到链表a
    let c = Cons(4, Box::new(a)); // 报错了，因为a的所有权已经被转移了
}
```

# 使用引用解决所有权问题

- 引用方式一：

```rust
use crate::List::{Cons, Nil};
#[derive(Debug)]
enum List<'a> {
    Cons(i32, Box<&'a List<'a>>),
    Nil,
}

fn main() {
    let a = Cons(5,
        Box::new(&Cons(10,
            Box::new(List::new(&Nil))) // 报错，创建临时引用&Nil，但是在a赋值时已经被销毁
        ) 
    ); // &Nil在这里的结束语句被销毁
    println!("{:?}", a);
}
```

这里涉及到生命周期的问题 `&Nil`是一个临时引用，在这里它的生命周期比a的生命周期要短。编译时就会报错了。

- 引用方式二：

```rust
use crate::List::{Cons, Nil};
#[derive(Debug)]
enum List<'a> {
    Cons(i32, Box<&'a List<'a>>),
    Nil,
}

fn main() {
    let a2 = &Nil;
    let a1 = &Cons(10, Box::new(a2));
    let a = &Cons(5, Box::new(a1));
    
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
    
    println!("{:?}", b); // Cons(3, Cons(5, Cons(10, Nil)))
    println!("{:?}", c); // Cons(4, Cons(5, Cons(10, Nil)))
}
```

虽然可以实现，但代码看上去太质朴，不够优雅，不能一眼看出嵌套关系，也就是不够直观。

# `Rc<T>` 登场

- 所有权转移引发的血案
- 使用引用又觉得不够优雅

`Rc<T>`来解决，它支持多重所有权，是`Reference counting`的缩写。它内部维护了一个引用次数计数器，用于确认这个值是否仍在使用。如果对一个值的引用次数为零，那么意味着这个值可以被安全清理掉了，而不会触发引用失效的问题：

```rust
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    // 替换Box为Rc
    Nil,
}

fn main() {
    // 创建Rc实例
    let a = Rc::new(Cons(5,
                         Rc::new(Cons(10,
                                      Rc::new(Nil)))));
    // 这里的Rc::clone只是增加引用计数，虽然使用a.clone也可以实现，但是数据会被深拷贝
    let b = Cons(3, Rc::clone(&a));

    // 再次增加引用次数
    let c = Cons(4, Rc::clone(&a));

    println!("{:?}", b);    // Cons(3, Cons(5, Cons(10, Nil)))
    println!("{:?}", c);    // Cons(4, Cons(5, Cons(10, Nil)))
}
```

# 观察引用计数

```rust
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    // 替换Box为Rc
    Nil,
}

fn main() {
    // 创建Rc实例
    let a = Rc::new(Cons(5,
                         Rc::new(Cons(10,
                                      Rc::new(Nil)))));
    println!("创建a之后的引用计数：{}", Rc::strong_count(&a)); // 1

    let b = Cons(3, Rc::clone(&a));
    println!("创建b之后胡引用计数：{}", Rc::strong_count(&a)); // 2

    // 进入一个代码块作用域
    {
        let c = Cons(4, Rc::clone(&a));
        println!("创建c之后的引用计数：{}", Rc::strong_count(&a)); // 3
    } // 离开代码块，c的作用域结束，c析构，a的引用计数会减1，变回到2

    println!("销毁c之后胡引用计数：{}", Rc::strong_count(&a)); // 2
}
```

# 总结

`Rc<T>`通过不可变引用使得程序的不同部分之间可以共享只读数据。如果`Rc<T>`允许持有多个可变引用的话，那么它就会违反借用规则：只允许多个不可变借用或一个可变借用。

