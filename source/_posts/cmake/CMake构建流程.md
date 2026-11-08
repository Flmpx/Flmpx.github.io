---
title: CMake构建流程
date: 2026-11-08 17:40:49
tags: cmake
---

构建器有 `Ninja` `Ninja Multi-Config` `MinGW Makefiles` `Unix Makefiles` 等等  

查看自己的cmake可以使用的构建器有哪些就输入命令 `cmake -G` 或者 `cmake --help | grep Generates` , 其中第一个会报错, 原因是第一个是要指定构建器但是你没指定  
```bash
flmpx@Flmpx:~/flmpx/learn_cmake$ cmake -G 
CMake Error: No generator specified for -G

Generators
  Green Hills MULTI            = Generates Green Hills MULTI files
                                 (experimental, work-in-progress).
* Unix Makefiles               = Generates standard UNIX makefiles.
  Ninja                        = Generates build.ninja files.
  Ninja Multi-Config           = Generates build-<Config>.ninja files.
... # 内容太多, 不展示了
```

> 建议使用 `Ninja` 或者 `Ninja Multi-Config` 构建器, 他们分别是单配置和多配置构建器, 且是跨平台的    


## 前置内容

> 目录树    

```txt
.
├── CMakeLists.txt
├── include
│   └── tools.hpp
├── main.cpp
└── src
    └── tools.cpp
```

 > CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.28.3)
project(lc)

# 设置标志
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/)

set(LC_MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)
file(GLOB LC_SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

add_executable(lc ${LC_SRC_LIST} ${LC_MAIN_SRC})

target_include_directories(lc PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/)
```

## 配置

```bash
cmake -G "Ninja" -B build -S .
```

`-G`  指定构建器, 如果没有指定, 就使用  
`-B`  指定构建目录, 构建产生的文件  
`-S`  指定项目的根目录(工作目录或者`CMakeLists.txt`所在目录), 一般为 `.` 或者不写(代表使用当前的工作目录)  

> 构建后的目录  

```txt
.
├── CMakeLists.txt
├── bin
├── build     # build/目录下增加了构建文件
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── build.ninja
│   ├── cmake_install.cmake
│   └── compile_commands.json
├── include
│   └── tools.hpp
├── main.cpp
└── src
    └── tools.cpp
```


## 构建

```bash
cmake --build build
```

`--build` 告诉构建器要构建的目录, 如果是 `Ninja` 构建器, 那这个等效于 `cd build && ninja`, 如果是 `Unix Makefiles`, 等效于 `cd build && make`

> 构建后的目录  

```txt
.
├── CMakeLists.txt
├── bin
│   └── lc     # 新增的可执行文件
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── build.ninja
│   ├── cmake_install.cmake
│   └── compile_commands.json
├── include
│   └── tools.hpp
├── main.cpp
└── src
    └── tools.cpp
```


## 运行

```bash
./bin/lc     # or `./bin/lc.exe` in windows
```


## 其他

如果使用的是 `Ninja` 这个单配置构建器的话, 当配置的时候指定了是生成Debug或者Release(或者其他的内容, 不然c++的标准)之后就不可以改变了, 除非删掉整个 `build/` 目录  
所以如果要同时生成多个版本, 就要使用 `Ninja Multi-Config` 这个构建器  

```bash
cmake -B build -S . -G "Ninja Multi-Config"
```

> 生成后的目录树

```txt
.
├── CMakeLists.txt
├── bin
│   ├── Debug     # debug产物目录 ------ 1
│   ├── RelWithDebInfo     # release版本但是带有debug调试信息的产物目录 ----- 2
│   └── Release      # release产物目录 ----- 3
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── build-Debug.ninja    # 1
│   ├── build-RelWithDebInfo.ninja     # 2
│   ├── build-Release.ninja     # 3
│   ├── build.ninja
│   ├── cmake_install.cmake
│   └── compile_commands.json
├── include
│   └── tools.hpp
├── main.cpp
└── src
    └── tools.cpp
```

这样之后就会将不同版本的内容生成在不同目录下  
