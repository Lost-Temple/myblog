---
title: Rust学习-知识点-2
tags:
  - Mutex
  - 多线程
  - Rc
  - Arc
cover: 'https://s2.loli.net/2022/10/14/bGPfeDVuNmsz7Ya.png'
categories:
  - Rust
  - 基础
abbrlink: 16222
date: 2022-10-14 09:15:52
---

# 在多个线程间共享`Mutex<T>`

我们启动10个线程，并在每个线程中分别为共享的计数器的值加1。如果正常执行完成，最终会让计数器的值从0累计到10：

```rust
use std::thread;
use std::sync::Mutex;

fn main() {
    //用于计数的互斥体
    let counter = Mutex::new(0);
    // 用于存储线程
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
          											// ^^^^^^^ value moved into closure here, in previous iteration of loop 
          	let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        // 等待所有线程都执行完毕
        handle.join().unwrap();
    }

    println!("counter: {}", *counter.lock().unwrap());
}
```

代码中尝试将`counter`移动到线程中，编译出错，因为第一次循环时，创建一个子线程，`counter`的所有权已经被移动进去，等到第二次循环创建第二个子线程，`counter`已经没有了所有权。这样就导致第二次移动时理论上肯定会失败。编译器很聪明地捕捉到了这个问题，在编译阶段就扼杀了这个`bug`。

# 尝试使用`Rc<T>`来共享counter

`Rc<T>`是用来共享数据用的，我们可以用尝试一下使用`Rc<T>`在多线程的场景下使用看看能不能解决之前遇到的所有权问题。`Rc<T>`是智能指针，它内部有一个引用计数，被它（包裹）指向的数据的引用次数被`Rc<T>`内部的引用计数记录。`Rc<T>`的`clone()`方法是对智能指针本身的复制，用来指向同一份被智能指针包裹的数据。这样就能巧妙规避数据的数据的所有权问题了：因为循环时可以每次`clone`一个新的智能指针`Rc<T>`出来（这个智能指针指向我们真正要使用的数据）。但是，事实上呢？编译器又来教做人了。

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    // 将Mutex再包裹一层Rc
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {        // 报错，Rc<Mutex<i32>>类型无法安全地在线程中传递
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("counter: {}", *counter.lock().unwrap())
}
```

当`Rc<T>`管理引用计数时，它会在每次调用`clone`的过程中增加引用计数，并在克隆出的实例被丢弃时减少引用计数，但它并没有使用任何并发原语来保证修改计数的过程不会被另一个线程所打断。

# 使用原子引用计数`Arc<T>`

rust还提供了`Arc<T>`类型，来代替`Rc<T>`类型来解决上面问题，它既拥有类似于`Rc<T>`的行为，又保证了自己可以被安全地用于并发场景：

```rust
use std::sync::Arc;
use std::sync::Mutex;
use std::thread;

fn main() {
    // 将Rc替换为Arc
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("counter: {}", *counter.lock().unwrap())
}
```

改用` Arc<T>`后上面成功计算出了` counter` 的值。

