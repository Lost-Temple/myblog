---
title: centos 多gcc版本环境
tags:
  - c++
  - cenos
  - CLion
  - Jetbrains gateway
  - 远程开发
cover: 'https://s2.loli.net/2022/11/30/vjtgMefxIDyu8EH.png'
categories: 
  - Linux
abbrlink: 42357
date: 2022-11-30 12:31:59
---
# 背景
- 在本地PC使用`Jetbrains gateway`新建ssh远程开发, 打开服务器上的代码进行开发
- CentOS 系统中的`gcc`和`g++`版本太低。不支持`c++11`标准，需要安装新的开发环境
- CLion 项目中的`CMakeList.txt`需要配置能编译`c++11标准`的`gcc`和`g++`
    
# 编译工具安装
```shell
    sudo yum install centos-release-scl-rh
    sudo yum install devtoolset-11-gcc-c++
```
> 安装完成后，执行以下命令可以切换到新的编译环境
```shell
    sudo scl enable devtoolset-11 bash
```
> 可以查看devtoolset-11相关工具的版本及它们的路径
```shell
    g++ --version
    gcc --version
```
```shell
    which g++
    which gcc
```

# 开发工具中的配置修改
> 需要指定编译器路径(**根据自己编译器路径进行修改**)
```
    SET(CMAKE_C_COMPILER "/opt/rh/devtoolset-11/root/usr/bin/gcc")
    SET(CMAKE_CXX_COMPILER "/opt/rh/devtoolset-11/root/usr/bin/g++")
```

# 调用动态库需要注意的点
> 在加载`so`库的模块相应的`.h`文件中要 #include <dlfcn.h>
> 在CMakefile.txt里面添加编译选项，如：
> TARGET_LINK_LIBRARIES(projName dl)
> 其中`projName`为项目的名称