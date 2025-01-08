---
title: C++ GUI--imgui库使用样板CMakefile
tags:
  - cmake
  - c++
  - cpp
  - mac
  - 跨平台
categories:
  - C++
  - GUI
cover: 'https://s2.loli.net/2022/12/08/VAJF2c3elutXjDM.png'
abbrlink: 47977
date: 2022-12-08 13:26:48
---

# imgui

先上链接：[github地址](https://github.com/ocornut/imgui)

`imgui`是一个轻量级，但是小而全的一个`cpp`界面库。作者应该是游戏开发行业的，这个库用来实现开发游戏还是比较适合的。虽然它是跨平台的，但是例子代码全是用`visual studio`的工程。在mac或linux下还不能做到完全的开箱即用。所以在网上搜索了一番，找到了一个[CMakeFile.txt](https://github.com/tashaxing/imgui_cmake_starter)可以跨平台的，我在`Mac`下尝试编译了一下，完全可以，非常感谢！

但是它的工程中使用的imgui不是最新的，我就拉取了一下imgui代码，替换掉它这个工程中的imgui，然后目录结构也稍作了一下调整。CMakeFile也稍相应地作了一下调整。

# CMakeFile.txt
```makefile
cmake_minimum_required(VERSION 3.0)

project(imgui_cmake_starter)

# add header path
include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}/3rd/imgui
	${CMAKE_CURRENT_SOURCE_DIR}/3rd/imgui/backends
)

if (APPLE)
    # for <GLFW/glfw3.h>
    include_directories(
        /usr/local/include
        /opt/local/include
        /opt/homebrew/include
    )
endif()

# set common source
file(GLOB SRC
    ./3rd/imgui/*.h
    ./3rd/imgui/*.cpp
)

# set specific source and other option for platform
if (WIN32)
    file (GLOB PLATFORM_SRC
        ./3rd/imgui/backends/imgui_impl_win32.*
        ./3rd/imgui/backends/imgui_impl_dx12.*
        ./src/win/main.cpp
    )
elseif (UNIX)
    # support both mac and linux
    add_definitions(-DIMGUI_IMPL_OPENGL_LOADER_GL3W)

    include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/3rd/imgui/examples/libs/gl3w # for GL/gl3w.h
    )

    file (GLOB PLATFORM_SRC
        ./3rd/imgui/examples/libs/gl3w/GL/gl3w.*
        ./3rd/imgui/backends/imgui_impl_glfw.*
        ./3rd/imgui/backends/imgui_impl_opengl3.*
        ./src/unix/main.cpp
    )
endif()

# add link path
if (APPLE)
    link_directories(
        /usr/local/lib
        /opt/homebrew/lib
#       这下面根据需要添加lib路径
#       /opt/local/lib
    )
endif()

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# generate binary
add_executable(${PROJECT_NAME} ${SRC} ${PLATFORM_SRC})

# link lib, should install glfw first or prebuild lib and embed in project
if (WIN32)
    target_link_libraries(${PROJECT_NAME}
        d3d12.lib
        d3dcompiler.lib
        dxgi.lib
    )
elseif (APPLE)
    # mac: brew install glfw3
    find_library(OPENGL_LIBRARY OpenGL REQUIRED)
    find_library(COCOA_LIBRARY Cocoa REQUIRED)
    find_library(IOKIT_LIBRARY IOKit REQUIRED)
    find_library(COREVID_LIBRARY CoreVideo REQUIRED)
    message(${COCOA_LIBRARY})
    message(${IOKIT_LIBRARY})
    message(${COREVID_LIBRARY})

    target_link_libraries(${PROJECT_NAME}
        ${OPENGL_LIBRARY}
        ${COCOA_LIBRARY}
        ${IOKIT_LIBRARY}
        ${COREVID_LIBRARY}
        glfw # use this lib name
    )
elseif (UNIX AND NOT APPLE)
    # linux: sudo apt install libglfw3-dev
    target_link_libraries(${PROJECT_NAME}
        GL
        glfw # use this lib name
        dl
    )
endif()

```

>以上的CMakeFile.txt文件内容我稍作调整：imgui我git pull了最新的代码下来，放在了工程目录下的3rd目录下面；我使用mac编译这个工程最到两个小问题，稍作记录

## 问题1
- 问题描述：我拉取的imgui版本使用的语法需要C++11标准的支撑，如果版本编译器的C++标准较低则会出现编译问题
- 解决办法：在CMakeFile.txt中指定C++标准
    ```
    # specify the C++ standard
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED True)
    ```

## 问题2
- 问题描述：我拉取的imgui代码目录`imgui/examples/libs/gl3w` 是不存在的，编译时`main.cpp`中以下语句会报错

    ```cpp

    #include <GL/gl3w.h>    

    ```
- 问题解决：缺什么补什么，就缺一个目录及几个文件，给它配齐就好

# 跨平台玩耍起来
以上部分已经完成了，[代码](https://github.com/Lost-Temple/imgui-starter)
要用imgui进行跨平台开发桌面应用啥的，直接拉取代码后当作个脚手架就可以开干了。
