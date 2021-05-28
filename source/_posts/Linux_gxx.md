---
title: gcc, g++ 使用切换 
date: 2021-03-15 12:01:01
tags: 
- Linux
- g++
- gcc
categories: 
- Linux
- g++
- gcc
top: 30 
description: 
password: 

---


# 安装版本
```
sudo apt-get install gcc-4.8 gcc-4.8-multilib g++-4.8 g++-4.8-multilib
```

<!--more-->

# 添加候选版本
```
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 100
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100
```
# 版本切换
```
sudo update-alternatives --config g++
sudo update-alternatives --config gcc
```
# 验证版本
```
jintao@localhost:~/soft/postgresql-9.1.6$ g++ -v
Using built-in specs.
COLLECT_GCC=g++
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/4.8/lto-wrapper
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 4.8.5-4ubuntu8' --with-bugurl=file:///usr/share/doc/gcc-4.8/README.Bugs --enable-		languages=c,c++,go,d,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-4.8 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.8 --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-gnu-unique-object --disable-libmudflap --enable-plugin --with-system-zlib --enable-objc-gc --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 4.8.5 (Ubuntu 4.8.5-4ubuntu8)

jintao@localhost:~/soft/postgresql-9.1.6$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/4.8/lto-wrapper
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 4.8.5-4ubuntu8' --with-bugurl=file:///usr/share/doc/gcc-4.8/README.Bugs --enable-languages=c,c++,go,d,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-4.8 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.8 --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-gnu-unique-object --disable-libmudflap --enable-plugin --with-system-zlib --enable-objc-gc --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 4.8.5 (Ubuntu 4.8.5-4ubuntu8)

```
