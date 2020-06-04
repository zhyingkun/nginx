# nginx

[![Build Status](https://travis-ci.com/zhyingkun/nginx.svg)](https://travis-ci.com/zhyingkun/nginx)

---

## 基本介绍

1. 专门为运行调试 Nginx 而构建的源码工程
2. 所有 Nginx 源码来自 [nginx 官方](https://nginx.org)1.18.0 版本
3. 仅保留 Mac/Linux 两个平台，去除了其他平台特定代码（为了足够简单）
4. 增加了源码注释，支持 Debug/Release 编译模式
5. 整个工程编译构建采用 cmake 来管理

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

---

## 文件夹说明

1. conf：nginx 默认配置文件
2. etc：nginx 官方源码中附带的一系列脚本、工具和文档
3. src：nginx 源码，加了注释

---

## 问题记录

1. 在 Mac 下使用 Nginx 自带的脚本执行 configure 时，需要提前配置 openssl 相关路径：

```bash
export CPATH=/usr/local/opt/openssl/include
export LIBRARY_PATH=/usr/local/opt/openssl/lib
```

2. 针对 Ubuntu 安装依赖：

```bash
sudo apt-get install libxslt1-dev -y
sudo apt-get install libpcre3-dev -y
sudo apt-get install libssl-dev -y
sudo apt-get install libgeoip-dev -y
sudo apt-get install libgd-dev -y
```

3. 针对 CentOS 安装依赖：

```bash
yum install -y openssl-devel.x86_64
yum install -y libxml2-devel.x86_64
yum install -y libxslt-devel.x86_64
yum install -y gd-devel.x86_64
yum install -y GeoIP-devel.x86_64
```
