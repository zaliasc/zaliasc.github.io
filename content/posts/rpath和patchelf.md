---
title: "rpath和patchelf"
date: 2020-07-02
draft: false
tags: ["Linux"]
# categories: ["ca1"]
---



rpath全称是run-time search path。Linux下所有elf格式的文件都包含它，特别是可执行文件。它规定了可执行文件在寻找.so文件时的第一优先位置。
 另外，elf文件中还规定了runpath。它与rpath相同，只是优先级低一些。



# 搜索.so的优先级顺序

- **编译目标代码时指定的动态库搜索路径； 如果在编译程序时增加参数-Wl,-rpath='.' , 这时生成程序的Dynamic section会新加一个RPATH段**
- **环境变量LD_LIBRARY_PATH指定的动态库搜索路径； ( 可用export LD_LIBRARY_PATH="NEWDIRS" 命令添加临时环境变量 )**
- **RUNPATH： 写在elf文件中**
- **ldconfig的缓存：配置文件/etc/ld.so.conf中指定的动态库搜索路径；(系统默认情况下未设置)**
- **默认的动态库搜索路径/lib；**
- **默认的动态库搜索路径/usr/lib；**

RPATH与RUNPATH中间隔着LD_LIBRARY_PATH，可以通过修改LD_LIBRARY_PATH来指定.so文件，大多数编译器都将输出的RPATH留空，并用RUNPATH代替RPATH。



# 查看RPATH

对于任意的elf文件，可以使用$ readelf -d filename | grep PATH 来查看。
0x000000000000001d (RUNPATH)            Library runpath: [$ORIGIN/]

结果有两类，一个是RPATH，另一个是RUNPATH。一般情况下，RPATH为空，而RUNPATH不为空。

RPATH中有个特殊的标识符$ORIGIN。这个标识符代表elf文件自身所在的目录。

当希望使用相对位置寻找.so文件，就需要利用$ORIGIN设置RPATH。多个路径之间使用冒号:隔开。



# 设置RPATH

在gcc中，设置RPATH的办法很简单，就是设置linker的rpath选项：

gcc -L. -larith main.c -Wl,-rpath='.'  -o main

（-Wl参数意思是把rpath选项传递到链接阶段）

如果需要设置$ORIGIN：$ gcc -Wl,-rpath,'$ORIGIN/lib' test.cpp。

注意，虽然选项里写着RPATH，但它设置的还是RUNPATH。原因在前文有交代。



在CMake中，使用变量来控制RPATH：INSTALL_RPATH和BUILD_RPATH。
设置的办法是：SET_TARGET_PROPERTIES(target PROPERTIES INSTALL_RPATH "$ORIGIN;/another/run/path")

（cmake中多个RPATH使用**分号**隔开）



## patchelf

patchelf 是一个用来修改elf格式的动态库和可执行程序的小工具，可以修改动态链接库的库名字，以及链接库的RPATH。

- 打印出动态库的soname  

  patchelf --print-soname xxx.so

- 修改动态库的soname

  patchelf --set-soname oldxxx.so newxxx.so

- 查看并修改第三方依赖库

  patchelf --print-needed xxx.so

  patchelf --replace-needed oldxxx.so newxxx.so this.so

- 修改rpath

  patchelf --set-rpath '$ORIGIN/' main