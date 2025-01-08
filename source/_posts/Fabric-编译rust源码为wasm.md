---
title: Fabric-编译rust源码为wasm
tags:
  - rust
  - wasm
  - chaincode
  - wasm-pack
  - build
cover: 'https://s2.loli.net/2022/11/04/8r6XMjU7ZIv9Ohp.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 55605
date: 2022-11-04 10:33:27
---

# 编译rust源码为wasm

## 安装工具链

可以使用以下两条命令：

```shell
curl https://sh.rustup.rs -sSf | sh
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

也可以参考[官方安装向导](https://rustwasm.github.io/book/game-of-life/setup.html)

## 创建`rust` `lib`工程

```shell
cargo new hello_world --lib
```

生成的`Cargo.toml`内容为：

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```

添加一些内容到`Cargo.toml`中去

```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

代码目录结构如下：

```
.
├── Cargo.toml
└── src
   └── lib.rs
```

在`lib.rs`中编写`rust`代码实现函数，因为在wasmcc的特殊性，这里的编写的代码也具有一些特殊性：

- 可以导入调用在wasmcc中实现的函数

  ```rust
  extern "C" {
      fn __print(msg: *const u8, len: usize) -> i64;
      fn __get_parameter(paramNumber: usize, result: *const u8) -> i64;
      fn __get_parameter_size(paramNumber: usize) -> i64;
      fn __get_state(msg: *const u8, len: usize, value: *const u8) -> i64;
      fn __get_state_size(msg: *const u8, len: usize) -> i64;
      fn __put_state(key: *const u8, key_len: usize, value: *const u8, value_len: usize) -> i64;
      fn __delete_state(msg: *const u8, len: usize) -> i64;
      fn __return_result(msg: *const u8, len: usize) -> i64;
  }
  ```

  **这些函数可以在编写`rust`代码时直接调用，不需要自己实现，这些是相当在Host程序中实现，可以被 `wasm`代码调用的函数**

  在rust的导出函数中,即wasm代码的对外接口中，可以调用上述的函数，如：

  - 通过__get_parameter_size(0)可以取得第1个参数的长度__
  - 通过__get_parameter可以取得参数的值，具体用法就是传入一个序号，一个vec!的指针

- 实现`init`函数（必须实现，`wasmcc`的`create`函数用来创建 wasm chaincode时就会调用wasm代码中的`init`函数

  ```rust
  #[no_mangle]
  pub extern "C" fn init(args: i64) -> i64 {
  	// ...
  }
  ```

## 编译

在`rust`工程的根目录下，运行命令：

```shell
wasm-pack build
```

如果成功的话，会在当前目录下生成`pkg/hello_world_bg.wasm`

关于C语言编写代码，再编译成wasm，文档中说是需要安装clang8 和 llvm。[文档](https://github.com/hyperledger-labs/fabric-chaincode-wasm/tree/main/sample-wasm-chaincode)中是[Linux的几个发行版的环境](https://apt.llvm.org/)，由于我的是Mac M1芯片的环境，尝试了一下得到一些链接出错信息，原因是找不到`__print`、`__get_parameter`等几个函数的找不到实现部分，所以链接的时候出错，这个理论上说得通，能编译成功才是不能理解的。不知道文档中说的方法是怎么编译成功的，到时候有空用Linux环境试一下吧。
