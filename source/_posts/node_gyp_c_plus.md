---
title:  linux下安装或升级GCC4.8，以支持C++11标准
category:  nodejs
tags: nodejs
---

C++11标准在2011年8月份获得一致通过，这是自1998年后C++语言第一次大修订，对C++语言进行了改进和扩充。随后各编译器厂商都各自实现或部分实现了C++中的特性。
linux服务器如果需要使用node的图片处理功能，很多组件需要C++11的支持
<!--more-->

　　本文主要介绍在Linux系统下，如何升级GCC以支持C++11。目前来看GCC是对C++11支持程度最高最多的编译器，但需要GCC4.8及以上版本。

 　  本文使用操作系统：Centos 6.4 Desktop，64bit；

　　原GCC版本：4.4.7；


1. 获取GCC 4.8.2包：wget http://gcc.skazkaforyou.com/releases/gcc-4.8.2/gcc-4.8.2.tar.gz；
2. 解压缩：tar -xf gcc-4.8.2.tar.gz；
3. 进入到目录gcc-4.8.2，运行：./contrib/download_prerequisites。这个神奇的脚本文件会帮我们下载、配置、安装依赖库，可以节约我们大量的时间和精力。
4. 建立输出目录并到目录里：mkdir gcc-build-4.8.2；cd gcc-build-4.8.2；
5. ../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib。--enable-languages表示你要让你的gcc支持那些语言，--disable-multilib不生成编译为其他平台可执行代码的交叉编译器。--disable-checking生成的编译器在编译过程中不做额外检查，也可以使用--enable-checking=xxx来增加一些检查；
6. 编译：make；注意此步和上一步，比较耗时；
7. 安装：make  install；
8. 验证：gcc -v；或者g++ -v，如果显示的gcc版本仍是以前的版本，就需要重启系统；或者可以查看gcc的安装位置：which gcc；然后在查看版本 /usr/local/bin/gcc -v，通常gcc都安装在该处位置，如果显示为；

