---
title: Rust学习-知识点-5
tags:
  - Result
  - Option
  - 错误传播
  - 错误处理
cover: 'https://s2.loli.net/2022/10/21/FOc73JrKDXPuL1x.jpg'
categories: 
  - Rust
  - 基础
abbrlink: 64799
date: 2022-10-19 15:22:44
---

# Rust中Trait的继承

`Trait`类似于`java`中的`interface`，可以继承

# Rust中的Struct的继承

~~**已被移除**~~

那怎么办呢？改为组合吧，文字描述总是苍白的，直接看代码吧

`inherit.rs`

```rust
use std::ops::{Deref, DerefMut};

pub struct Dog<'a> {
    name: &'a str,
}

impl<'a> Dog<'a> {
    pub fn new(name: &'a str) -> Dog {
        Dog {
            name
        }
    }
    pub fn run(&self) {
        println!("the dog is running.")
    }

    pub fn sleep(&self) {
        println!("the dog is sleeping.")
    }

    pub fn bark() {
        println!("the dog is barking.")
    }
}

pub struct Husky<'a> {
    dog: Dog<'a>,
}

impl<'a> Husky<'a> {
    pub fn new(name: &'a str) -> Husky<'a> {
        Husky {
            dog: Dog::new(name)
        }
    }
    pub fn get_dog(&'a self) -> &Dog<'a> {
        &self.dog
    }
    pub fn run(&self) {
        println!("the husky is running.");
    }
    pub fn sleep(&self) {
        println!("the husky is sleeping.");
    }
    pub fn do_stupid_thing(&self) {
        println!("the husky is acting as a fool.")
    }
}

impl<'a> Deref for Husky<'a> {
    type Target = Dog<'a>;

    fn deref(&self) -> &Self::Target {
        &self.dog
    }
}

impl<'a> DerefMut for Husky<'a> {
    fn deref_mut(&mut self) -> &mut Dog<'a> {
        &mut self.dog
    }
}
```

调用端代码：`main.rs`

```rust
mod inherit;

use inherit::{Dog, Husky};

fn main() {
    let husky = Husky::new("fool");
    husky.sleep(); // the husky is sleeping.
    husky.run(); // the husky is running.
    husky.do_stupid_thing(); // the husky is acting as a fool.

    let mut husky_dog: &Dog = &husky;
    husky.get_dog().sleep(); // the dog is sleeping.
    husky_dog.sleep(); // the dog is sleeping.
    husky_dog.run(); // the dog is running.
}
```





