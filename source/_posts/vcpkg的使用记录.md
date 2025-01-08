---
title: vcpkg的使用记录
categories:
  - C++
  - 开发环境
tags:
  - vcpkg
  - boost
  - CMake
abbrlink: 18533
date: 2024-04-24 08:24:30
cover: 'https://s2.loli.net/2022/12/08/VAJF2c3elutXjDM.png'
---

# 安装vcpkg

## clone vcpkg
```shell
git clone https://github.com/microsoft/vcpkg
```

## install vcpkg

```shell
cd vcpkg
./bootstrap-vcpkg.sh
```

## 设置环境变量

```shell
# vcpkg
export VCPKG_ROOT=~/vcpkg
export PATH=$PATH:$VCPKG_ROOT
export VCPKG_DEFAULT_TRIPLET=arm64-osx
```

> VCPKG_DEFAULT_TRIPLET 设置了默认的TRIPLET，vcpkg install 时就无需指定triplet

# vcpkg install

## 安装boost

```shell
vcpkg install boost
```

在最后会出错，有警告信息如下：

```shell
CMake Warning at ports/python3/portfile.cmake:7 (message):
  python3 currently requires the following programs from the system package
  manager:

      autoconf automake autoconf-archive

  On Debian and Ubuntu derivatives:

      sudo apt-get install autoconf automake autoconf-archive

  On recent Red Hat and Fedora derivatives:

      sudo dnf install autoconf automake autoconf-archive

  On Arch Linux and derivatives:

      sudo pacman -S autoconf automake autoconf-archive

  On Alpine:

      apk add autoconf automake autoconf-archive

  On macOS:

      brew install autoconf automake autoconf-archive
```

> 按照上述警告信息操作后再重新执行vcpkg install boost，成功

## 安装nlohmann-json

```shell
vcpkg install nlohmann-json
```

# 创建c++项目

### vcpkg 创建项目

```shell
mkdir helloworld && cd helloworld
vcpkg new --application
```

> 会生成vcpkg.json文件，项目中的依赖库信息会保存在里面

### 添加依赖

```shell
vcpkg add port nlohmann-json
```

执行后在`vcpkg.json`文件中会添加这个依赖：

```json
{
  "dependencies": [
    "nlohmann-json"
  ]
}
```

## 创建`CMakeLists.txt`文件

```shell
cmake_minimum_required(VERSION 3.10)
project(HelloWorld)
add_executable(HelloWorld helloworld.cpp)
find_package(nlohmann_json CONFIG REQUIRED)
target_link_libraries(HelloWorld PRIVATE nlohmann_json::nlohmann_json)
```

> `CMakeLists.txt`中并没有指定nlohmann_json包的路径，那CMake怎么找到这些依赖包呢？那就要用到`CMakePresets.json`文件

## 创建`CMakePresets.json`文件

```json
{
  "version": 2,
  "configurePresets": [
    {
      "name": "default",
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
      }
    }
  ]
}
```

> 这里就需要在环境变量中设置`VCPKG_ROOT`

有了`CMakePresets.json`后，使用以下命令，会对cmake build 做一下一些前置的设置操作：

```shell
cmake --preset=default
```

## 编译

```shell
cmake --build build
```

