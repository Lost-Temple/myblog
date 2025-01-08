---
title: ABY开源框架编译
tags:
  - fedora
  - cmake
  - boost
cover: 'https://s2.loli.net/2022/12/08/VAJF2c3elutXjDM.png'
abbrlink: 60236
date: 2023-06-11 18:20:53
---

# ABY开源框架
[ABY开源框架](https://github.com/encryptogroup/ABY)

# 编译准备

- cmake

  ```shell
  sudo dnf install cmake
  ```
  
- C Development Tools and Libraries

  ```shell
  sudo dnf group install "C Development Tools and Libraries" "Development Tools"
  ```
- boost

  ```shell
  sudo dnf install boost-devel
  ```
  
- libgmp

  ```shell
  sudo dnf install gmp-devel
  ```
  
- libssl

  ```shell
  sudo dnf install openssl-devel
  ```

# 编译

- git clone

  ```shell
  git clone https://github.com/encryptogroup/ABY.git
  ```
  
- cd

  ```shell
  cd ABY/
  ```
- mkdir

  ```shell
  mkdir build && cd build
  ```
  
- cmake && make && make install

  ```shell
  cmake .. -DCMAKE_INSTALL_PREFIX=""
  make
  make DESTDIR=~/path/to/aby/prefix install
  ```
  
  or

  ```shell
  cmake .. -DCMAKE_INSTALL_PREFIX=~/path/to/aby/prefix
  make
  make install
  ```
  
> 这一步我出错了。编译报错文件：`ABY/extern/ENCRYPTO_utils/src/ENCRYPTO_utils/channel.cpp`
> 错误原因就是缺少了`#include <cstdlib>`,引入头文件后就好了

# 编译选项

```shell
cmake .. -DCMAKE_BUILD_TYPE=Release
# or
cmake .. -DCMAKE_BUILD_TYPE=Debug
```

