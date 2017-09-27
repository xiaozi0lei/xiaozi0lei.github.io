---
layout: default
title:  "MacOS 10.12.6 编译 openJDK7"
date:   2017-09-27 08:53:51 +0800
categories: java
---

## 安装XQuartz
地址 https://www.xquartz.org/

## 安装CUPS
地址 https://github.com/apple/cups/releases
```bash
tar zxvf cups-2.2.4-source.tar.gz
cd cups-2.2.4
./configure --prefix=/usr/local/cups
make
sudo make install
```

## 做软链接，需要关闭 rootLess 功能
* sudo ln -s `type ant` /usr/bin/ant
* sudo ln -s `type gcc` /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-gcc
* sudo ln -s `type g++` /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-g++
* sudo ln -s /usr/X11/include/X11 /usr/include/X11

## 下载 bootstrap java 和 openjdk7 源码
* 编译的源码用的是 __openjdk-7u6-fcs-src-b24-28_aug_2012.zip__，地址为 http://download.huihoo.com/java/src/  或者   https://pan.baidu.com/s/1pJ4rxyV (该源文件中 hotspot 下有一个 CPrinterDialogPeer.cpp 文件是乱码，需要替换一个正确的文件)
* Bootstrap java 用的 MacOS 官方的 jdk6，地址为 https://support.apple.com/kb/DL1572?locale=zh_CN

## 环境变量
```bash
#!/bin/bash

#设置语言
export LANG=C
#允许自动下载依赖包
export ALLOW_DOWNLOADS=true

#使用预编译头文件，不加这个编译会更慢
export USE_PRECOMPILED_HEADER=true

#要编译的内容
export BUILD_LANGTOOLS=true
export BUILD_JAXP=true
export BUILD_JAXWS=true
export BUILD_CORBA=true
export BUILD_HOSTPOT=true
export BUILD_JDK=true

#要编译的版本
export SKIP_DEBUG_BUILD=false
export SKIP_FASTDEBUG_BUILD=true
export DEBUG_NAME=debug

#把它设置为FALSE可以避免javaws和浏览器Java插件之类的部分build
export BUILD_DEPLOY=false

#把它设置为false就不会build出安装包。因为安装包里有一些奇怪的依赖
#但即便不build出它也已经得到完整的JDK镜像,所以还是不用build它
export BUILD_INSTALL=false

#存放编译结果
export ALT_OUTPUTDIR=/Users/guolei/webtool/openjdk

unset CLASSPATH
unset JAVA_HOME
unset LD_LIBRARY_PATH

#make debug_build 2>&1 | tee $ALT_OUTPUTDIR/build.log

export ALT_BOOTDIR=`/usr/libexec/java_home -v 1.6`

export COMPILER_WARNINGS_FATAL=false
export CC=clang
export COMPILER_WARNINGS_FATAL=false
export USE_CLANG=true
export LP64=1
export ARCH_DATA_MODEL=64
export LFLAGS='-Xlinker -lc++ -lstdc++'
export HOTSPOT_BUILD_JOBS=8
export SHOW_ALL_WARNINGS=false
export INCREMENTAL_BUILD=true

export _JAVA_OPTIONS=-Dfile.encoding=ASCII
```

## 执行编译前检查
`make sanity`

## 编译过程遇到的问题，按先后顺序解决
执行编译命令 `make debug_build 2>&1 | tee $ALT_OUTPUTDIR/build.log`，在每一步报错之后，查看 `$ALT_OUTPUTDIR/build.log`
* 乱码报错

```bash
# 解决办法
cd /Users/guolei/webtool/openjdk-debug

find corba/gensrc/org/ -name '*.java' | while read p; do native2ascii -encoding UTF-8 $p > tmpj; mv tmpj $p; done
```
* 形参默认值问题

```bash
# 报错信息
openjdk/hotspot/src/share/vm/code/relocInfo.hpp:374:27: error: friend declaration specifying a default argument must be a definition
# 解决办法
# 修改relocInfo.hpp（路径：hotspot/src/share/vm/code/relocInfo.hpp）
# 修改374行
inline friend relocInfo prefix_relocInfo(int datalen);
# 修改469行
inline relocInfo prefix_relocInfo(int datalen = 0) {
   assert(relocInfo::fits_into_immediate(datalen), "datalen in limits");
   return relocInfo(relocInfo::data_prefix_tag, relocInfo::RAW_BITS, relocInfo::datalen_tag | datalen);
}
```
* 错误 openjdk/hotspot/src/share/vm/opto/loopPredicate.cpp:834:73: error: ordered comparison between pointer and zero ('const TypeInt *' and 'int')

```bash
# 解决办法
# 修改 loopPredicate.cpp （路径：hotspot/src/share/vm/opto/loopPredicate.cpp）
# 修改 834 行，替换为
assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int()->_lo >= 0, "must be");
```
* 十年问题 Error: time is more than 10 years from present: 1136059200000

```bash
# 解决办法
# 需要修改jdk/src/share/classes/java/util/CurrencyData.properties,将类似AZ=AZM;2005-12-31-20-00-00;AZN的时间修改为距离编译日期小于10年，比如我统一修改为2015-12-31-20-00-00;
# 修改CurrencyData.properties（路径：jdk/src/share/classes/java/util/CurrencyData.properties）
修改108行
AZ=AZM;2009-12-31-20-00-00;AZN
修改381行
MZ=MZM;2009-06-30-22-00-00;MZN
修改443行
RO=ROL;2009-06-30-21-00-00;RON
修改535行
TR=TRL;2009-12-31-22-00-00;TRY
修改561行
VE=VEB;2009-01-01-04-00-00;VEF
```
* 非空函数不返回值的问题一

```bash
# 具体报错
../../../src/solaris/native/java/net/net_util_md.c:117:9: error: non-void function 'getDefaultScopeID' should return a value [-Wreturn-type]
        CHECK_NULL(c);
        ^
../../../src/share/native/java/net/net_util.h:45:40: note: expanded from macro 'CHECK_NULL'
#define CHECK_NULL(x) if ((x) == NULL) return;

# 解决办法
# 修改net_util_md.c（路径：jdk/src/solaris/native/java/net/net_util_md.c）
修改117、119行
CHECK_NULL_RETURN(c , 0);
```
* 权限问题

```bash
# 具体报错
Permission denied - ./src/core/PrimitiveCoder.hs (Errno::EACCES)

# 解决办法
chmod 755 jdk/src/macosx/native/jobjc/src/core/PrimitiveCoder.hs
```
* /Users/guolei/github/openjdk/jdk/src/macosx/native/jobjc/src/core/native/SEL.m:37:12: error: cast of type 'SEL' to 'uintptr_t' (aka 'unsigned long') is deprecated; use sel_getName instead [-Werror,-Wcast-of-sel-type]

```
# 解决办法
# 修改 SEL.m 进行强制 cast，将37行替换为
return ptr_to_jlong((void *)sel);
```
* 返回值大于返回类型精度的问题

```bash
# 具体报错
/Users/guolei/webtool/openjdk-debug/JObjC.build/src/jobjc/com/apple/jobjc/appkit/AppKitFramework.java:444: ?????
    [javac]     public final float NSEventDurationForever(){ return 1.797693134862316E+308f; }

# 解决办法
1. 修改bridgesupport.gmk（路径：jdk/src/macosx/native/jobjc/bridgesupport.gmk）
修改54行，替换为
all:

2. 修改AppKitFull.bridgesupport（路径：openjdk-debug/stable_bridge_metadata/AppKitFull.bridgesupport）
cd /Users/guolei/webtool/openjdk-debug
vim stable_bridge_metadata/AppKitFull.bridgesupport
修改1692行
<enum name='NSEventDurationForever' value='3.40282E+38'/>
```

## 编译完成

```bash
########################################################################
##### Leaving jdk for target(s) sanity all  images                 #####
########################################################################
##### Build time 00:20:36 jdk for target(s) sanity all  images     #####
########################################################################

#-- Build times ----------
Target debug_build
Start 2017-09-27 13:38:53
End   2017-09-27 14:00:13
00:00:15 corba
00:00:16 hotspot
00:00:04 jaxp
00:00:05 jaxws
00:20:36 jdk
00:00:04 langtools
00:21:20 TOTAL
-------------------------
bogon:openjdk guolei$
bogon:j2sdk-image guolei$ ./bin/java -version
openjdk version "1.7.0-internal-debug"
OpenJDK Runtime Environment (build 1.7.0-internal-debug-guolei_2017_09_27_11_36-b00)
OpenJDK 64-Bit Server VM (build 23.2-b09-jvmg, mixed mode)
bogon:j2sdk-image guolei$
```