---
layout: article
title: NDK r21d + x264 编写脚本文件 & 编译生成 Android 所需的库
date: 2020-09-18 0:19:14
tags:
categories: 
copyright: true
---

# **下载 x264 源码**
```shell
git clone https://code.videolan.org/videolan/x264.git
```

---

# **编写脚本文件**

## **查看文件目录**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/1.png)

## **查看支持的命令**
```shell
./configure --help
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/2.png)

## **部分命令说明**
Standard options|描述
--|--
--prefix=PREFIX          |指定生成的文件存放的位置
--exec-prefix=EPREFIX    |
--bindir=DIR             |
--libdir=DIR             |[PREFIX/lib]
--includedir=DIR         |[PREFIX/include]
--extra-asflags=EASFLAGS |
--extra-cflags=ECFLAGS   |本质是设置 CFLAGS 变量（默认 gcc、clang 会读取 CFLAGS 变量）
--extra-ldflags=ELDFLAGS |
--extra-rcflags=ERCFLAGS |

Configuration options|描述
--|--
--disable-cli            |cli: 命令行工具
--system-libx264         |
--enable-shared          |动态库，so 文件
--enable-static          |静态库，a 文件
--disable-bashcompletion |
--enable-bashcompletion  |
--bashcompletionsdir=DIR |
--disable-opencl         |
--disable-gpl            |
--disable-thread         |
--disable-win32thread    |
--disable-interlaced     |
--bit-depth=BIT_DEPTH    |
--chroma-format=FORMAT   |

Advanced options|描述
--|--
--disable-asm            |
--enable-lto             |
--enable-debug           |本质是在 CFLAGS 添加 -g
--enable-gprof           |本质是在 CFLAGS 添加 -pg
--enable-strip           |本质是在 CFLAGS 添加 -s
--enable-pic             |*本质是在 CFLAGS 添加 -fPIC，Android 需要

Cross-compilation|描述
--|--
--host=HOST              |*运行的平台
--cross-prefix=PREFIX    |*设置库文件的路径前缀
--sysroot=SYSROOT        |*指定库与头文件的搜索路径的根目录（默认会使用 Linux 的文件，Android 需要改为使用 NDK 中的文件）

External library support|描述
--|--
--disable-avs            |
--disable-swscale        |
--disable-lavf           |
--disable-ffms           |
--disable-gpac           |
--disable-lsmash         |

## **编写脚本文件**
1、创建文件
```shell
gedit build.sh
```

* CC: 指定编译 C 的编译器，需要以环境变量使用
* CXX: 指定编译 C++ 的编译器，需要以环境变量使用

```shell
#!/bin/bash
 
function my_build
{
./configure \
--prefix=$MY_PREFIX \
--extra-cflags=$MY_EXTRA_CFLAGS \
--disable-cli \
--enable-shared \
--enable-pic \
--host=$MY_HOST \
--cross-prefix=$MY_CROSS_PREFIX \
--sysroot=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/sysroot \

make clean
make -j8
make install
}
 
#arm64-v8a
export CC=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang
export CXX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang++
MY_PREFIX=./Android/arm64-v8a
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=aarch64-none-linux-android21 --gcc-toolchain=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64 $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=aarch64-linux-android
MY_CROSS_PREFIX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android-
my_build
 
#armeabi-v7a
export CC=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi21-clang
export CXX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi21-clang++
MY_PREFIX=./Android/armeabi-v7a
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -march=armv7-a -mthumb -Wformat -Werror=format-security   -Oz -DNDEBUG  -fPIC --gcc-toolchain=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64 $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=armv7a-linux-android
MY_CROSS_PREFIX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/arm-linux-androideabi-
my_build
 
#x86_64
export CC=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/x86_64-linux-android21-clang
export CXX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/x86_64-linux-android21-clang++
MY_PREFIX=./Android/x86_64
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=x86_64-none-linux-android21 --gcc-toolchain=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64 $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=x86_64-linux-android
MY_CROSS_PREFIX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/x86_64-linux-android-
my_build
 
#x86
export CC=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android24-clang
export CXX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android24-clang++
MY_PREFIX=./Android/x86
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -mstackrealign -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=i686-none-linux-android21 --gcc-toolchain=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64 $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=i686-linux-android
MY_CROSS_PREFIX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android-
my_build
```

### **添加 Android 项目需要的 CFLAGS 的技巧**
当 configure 不支持某个参数时，需要主动在 CFLAGS 中添加。

1、新建 Android 的 C++ 项目

2、build release 版本，生成 build.ninja、rules.ninja
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/3.png)

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/4.png)

3、使用 build.ninja 中的 FLAGS
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/5.png)

```
-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -march=armv7-a -mthumb -Wformat -Werror=format-security   -Oz -DNDEBUG  -fPIC
```

4、使用 rules.ninja 中的参数
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/6.png)

```shell
--target=armv7-none-linux-androideabi21 --gcc-toolchain=/home/weichao/android-sdk/ndk/21.3.6528147/toolchains/llvm/prebuilt/linux-x86_64 --sysroot=/home/weichao/android-sdk/ndk/21.3.6528147/toolchains/llvm/prebuilt/linux-x86_64/sysroot  $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in
```

调整路径，修改参数，去掉重复的参数：
```shell
--target=armv7-none-linux-androideabi21 --gcc-toolchain=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64 $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in
```

## **对脚本中的变量进行抽取，方便修改**
```shell
#!/bin/bash

NDK_ROOT=/home/weichao/android-ndk-r21d
TOOLCHAIN=$NDK_ROOT/toolchains/llvm/prebuilt/linux-x86_64
API=21
API_2=24 #x86在24以下会报错，所以单独指定

function my_build
{
./configure \
--prefix=$MY_PREFIX \
--extra-cflags=$MY_EXTRA_CFLAGS \
--disable-cli \
--enable-shared \
--enable-pic \
--host=$MY_HOST \
--cross-prefix=$MY_CROSS_PREFIX \
--sysroot=$TOOLCHAIN/sysroot \

make clean
make -j8
make install
}

#arm64-v8a
export CC=$TOOLCHAIN/bin/aarch64-linux-android$API-clang
export CXX=$TOOLCHAIN/bin/aarch64-linux-android$API-clang++
MY_PREFIX=./Android/arm64-v8a
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=aarch64-none-linux-android21 --gcc-toolchain=$TOOLCHAIN $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=aarch64-linux-android
MY_CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
my_build

#armeabi-v7a
export CC=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang
export CXX=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang++
MY_PREFIX=./Android/armeabi-v7a
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -march=armv7-a -mthumb -Wformat -Werror=format-security   -Oz -DNDEBUG  -fPIC --gcc-toolchain=$TOOLCHAIN $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=armv7a-linux-android
MY_CROSS_PREFIX=$TOOLCHAIN/bin/arm-linux-androideabi-
my_build

#x86_64
export CC=$TOOLCHAIN/bin/x86_64-linux-android$API-clang
export CXX=$TOOLCHAIN/bin/x86_64-linux-android$API-clang++
MY_PREFIX=./Android/x86_64
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=x86_64-none-linux-android21 --gcc-toolchain=$TOOLCHAIN $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=x86_64-linux-android
MY_CROSS_PREFIX=$TOOLCHAIN/bin/x86_64-linux-android-
my_build

#x86
export CC=$TOOLCHAIN/bin/i686-linux-android$API_2-clang
export CXX=$TOOLCHAIN/bin/i686-linux-android$API_2-clang++
MY_PREFIX=./Android/x86
MY_EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -mstackrealign -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=i686-none-linux-android21 --gcc-toolchain=$TOOLCHAIN $DEFINES $INCLUDES $FLAGS -MD -MT $out -MF $DEP_FILE -o $out -c $in"
MY_HOST=i686-linux-android
MY_CROSS_PREFIX=$TOOLCHAIN/bin/i686-linux-android-
my_build
```

---

# **编译生成 Android 所需的库**

## **给脚本文件设置权限**
```shell
chmod +x build.sh
```

## **执行脚本文件**
```shell
. build.sh
```

编译可能会遇到缺少 NASM 库的情况，下载安装比较新的版本就可以了。

可以看到 CFLAGS 中的不识别的参数会被忽略：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/7.png)

然后展示基本信息：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/8.png)

然后是详细过程，最后生成文件：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/9.png)

## **编译完成后生成文件**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/10.png)

## **编译结果查看 log**
/x264/config.log

### **编译 x86 版本时报错：clang: error: unknown argument: '-mpreferred-stack-boundary=6'**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK-r21d-x264-%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-Android-%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/11.png)

原因貌似是当 clang 遇到不识别的参数时会报 error，是 clang 的处理机制问题。参见[Issue 475183: clang: does not support -mpreferred-stack-boundary=2](https://bugs.chromium.org/p/chromium/issues/detail?id=475183 "https://bugs.chromium.org/p/chromium/issues/detail?id=475183")。

解决办法有 2 个：
1、去掉不识别的参数。
2、使用更高版本的修复了这个问题的 clang。

由于我没找到这个参数是在哪加的，且不清楚去掉后会有什么影响，所以我采用第 2 种方法解决这个问题，最终在 i686-linux-android24-clang 版本解决了问题。
```shell
export CC=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android24-clang
export CXX=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android24-clang++
```

---