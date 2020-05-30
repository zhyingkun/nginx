# nginx

[![Build Status](https://travis-ci.com/zhyingkun/nginx.svg)](https://travis-ci.com/zhyingkun/nginx)

---

## 基本介绍

1. 专门为运行调试 Nginx 而构建的源码工程
2. 所有 Nginx 源码来自 [nginx 官方](https://nginx.org)1.18.0 版本
3. 仅保留 Mac/Linux/Win 三个平台的源码，去除了其他平台特定代码（为了足够简单）
4. 增加了源码注释，支持 Debug/Release 编译模式
5. 整个工程编译构建采用 cmake 来管理，支持跨平台（可以在树莓派上正常 cmake+make）

---

## 如何编译

#### 1. 在 Mac 上采用 Xcode 编译

```bash
cd nginx/
mkdir buildXcode && cd buildXcode
cmake -DCMAKE_INSTALL_PREFIX=./install -G "Xcode" ..
# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/zyk/nginx -G "Xcode" ..
```

此时已经在 buildXcode 文件夹下生成了 Xcode 工程，直接打开并编译即可

#### 2. 直接命令行编译（支持 Mac 和 Linux）

```bash
cd nginx/
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=./install .. # default is Debug
# for Debug: cmake -DCMAKE_BUILD_TYPE=Debug ..
# for Release: cmake -DCMAKE_BUILD_TYPE=Release ..
# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/zyk/nginx -DCMAKE_BUILD_TYPE=Release ..
make
# for more details: make VERBOSE=1
make install
```

make 命令会自动编译好各个模块

#### 3. 在 Windows 上使用 Visual Studio 2017 进行编译

```bash
# In Cygwin
cd nginx/
mkdir buildVS && cd buildVS
cmake -DCMAKE_INSTALL_PREFIX=./install -G "Visual Studio 15 2017 Win64" ..
# cmake -DCMAKE_INSTALL_PREFIX=D:/Applications/zyk/nginx -G "Visual Studio 15 2017 Win64" ..
```

非 Cygwin 可以使用 GUI 版本的 CMake 来生成 Visual Studio 工程

在 buildVS 文件夹下生成了 Visual Studio 工程后，双击打开并编译 ALL_BUILD 目标

Windows 环境下需要将 CMAKE_INSTALL_PREFIX 设置路径下 bin 文件夹加入系统 path 环境变量，以便在 cmd 命令行中能调用到

---

## 文件夹说明

1. conf：nginx 默认配置文件
2. etc：nginx 官方源码中附带的一系列脚本、工具和文档
3. src：nginx 源码，加了注释
