---
title: "linux链接相关"
date: 2020-07-02
draft: false
tags: ["Linux"]
# categories: ["ca1"]
---



程序函数库可分为3种类型：静态函数库（static libraries）、共享函数库（shared libraries）、动态加载函数库（dynamically loaded libraries）： 

- 静态函数库，是在程序执行前就加入到目标程序中去了；

- 共享函数库，则是在程序启动的时候加载到程序中，它可以被不同的程序共享；动态加载函数库则可以在程序运行的任何时候动态的加载。

- 动态函数库，并非另外一种库函数格式，区别是动态加载函数库是如何被程序员使用的。 



GCC编译基础流程包括：预处理、编译、汇编、链接。



## 静态链接

- 链接器在链接时将库的内容加入到可执行程序中

- 静态函数库实际上就是简单的一个普通的目标文件的集合，一般来说习惯用“.a”作为文件的后缀。可以用ar这个程序来产生静态函数库文件

- 静态函数库现在已经不在像以前用得那么多了，主要是共享函数库与之相比较有很多的优势的原因。

- 静态库函数允许把程序link起来而不用重新编译代码，节省了重新编译代码的时间。

- 把自己提供的函数给别人使用，但是又想对函数的源代码进行保密，就可以提供一个静态函数库文件。

### 静态函数库（静态链接库）：程序编译时使用

　　扩展名：.a（archive）

　　命名格式：libxxx.a

　　独立执行：编译成功的可执行文件可以独立执行

### demo：

```
./lib/sqrt.h
extern int sqrt(int i);

./lib/sqrt.cpp
#include"sqrt.h"
int sqrt(int i ){return i*i;}

#gcc -c sqrt.cpp
#ar rcs libmylib.a sqrt.o
#注意命名方式 以lib开头 .a结束

./"test.cpp"
#include<iostream>
extern int sqrt(int i);
int main(){
    std::cout << sqrt(100) <<std::endl ;
    return 0;
}

#g++ -o test.cpp -L ./lib -l mylib
#当libmylib.c在同一目录下时 直接 g++ -o test.cpp libmylib.a 

上述编译过程都没有加入-static选项，所以默认都是采用动态链接（dynamically linked）。加上-static后才是静态链接（statically linked）的。用静态库既可以实现静态链接也可以实现动态链接。
```

## 动态链接

- 链接器在链接时仅仅建立与所需库函数之间的链接关系，在程序运行时才将所需资源调入可执行程序中

- 动态链接库，在Linux下是.so文件，在编译链接时只需要记录需要链接的号，运行程序时才会进行真正的“链接”，所以称为“动态链接”。

- 如果同一台机器上有多个服务使用同一个动态链接库，则只需要加载一份到内存中共享。因此，动态链接库也称共享库。

### 命名规则

- 动态链接库与应用程序之间的真正链接是在应用程序运行时，因此很容易出现开发环境和运行环境的动态链接库不兼容或缺失的情况。

- Linux通过规定动态链接库的版本命名规则来管理兼容性问题。

- Linux规定动态链接库的文件名规则比如如下：

                           libname.so.x.y.z

	- lib：统一前缀。

	- so：统一后缀。

	- name：库名，如libstdc++.so.6.0.21的name就是stdc++。

	- x：主版本号。表示库有重大升级，不同主版本号的库之间是不兼容的。如libstdc++.so.6.0.21的主版本号是6。

	- y：次版本号。表示库的增量升级，如增加一些新的接口。在主版本号相同的情况下，高的次版本号向后兼容低的次版本号。如libstdc++.so.6.0.21的次版本号是0。    

	- z：发布版本号。表示库的优化、bugfix等。相同的主次版本号，不同的发布版本号的库之间完全兼容。如libstdc++.so.6.0.21的发布版本号是21。

	另外，Linux下的一个动态链接库会有下面三个名字：

	libstdc++.so -> libstdc++.so.6.0.21* libstdc++.so.6 -> libstdc++.so.6.0.21* libstdc++.so.6.0.21*

	libstdc++.so：linker name，程序编译链接时如果依赖了共享库，链接器只认不带任何版本的共享库。如果存在多个同名（上面命名规则中的name）动态链接库，linker name会指向最新的一个。

	libstdc++.so.6：SO_NAME， 程序运行时会按照这个名称去找真正的库文件。也就是说，ELF可执行文件中保存的动态库名就是SO_NAME。如果存在多个同一主版本号的动态链接库，SO_NAME会指向最新的一个。

	libstdc++.so.6.0.21：real name，这是动态链接库的真正名称。

### 动态函数库（动态链接库）：程序运行时使用

　　扩展名：.so (shared object)

　　命名格式：libxxx.so

　　独立执行：编译成功的可执行文件不可独立执行，库函数文件必须要存在，并且函数库所在目录也不能改变

注：使用gcc hello.c -o hello时，系统默认使用动态链接方式进行编译程序，若想使用静态编译，需加入-static参数

　　如果静态库、动态库都放在/lib下，当静态库函数与动态库函数重名时，系统也是优先考虑链接动态库，同样如果想要使用静态库，需加入-static选项

demo:

```
./"arith.h"
#pragma once
int add(int a, int b);


./"arith.c”
#include "arith.h"
int add(int a, int b)
{
    return a + b;
}

#gcc -fPIC -shared arith.cpp -o libarith.so

./"main.cpp"
#include "arith.h"
int main()
{
    add(1, 2);
}

如果我们不指定rpath直接编译main.c：
$ gcc -L. -larith main.c -o main

若此时运行main文件会报如下错误：
./main: error while loading shared libraries: libarith.so: cannot open shared object file: No such file or directory

报错提示找不到libarith.so文件。该文件在我们当前目录下，但是当前目录并不在运行时链接器的搜索路径中。一种解决办法是将当前路径添加到LD_LIBRARY_PATH中，但是该方法是一种全局配置，总是显得不那么干净。下面介绍第二种方法，就是在链接的时候直接将搜索路径写到RPATH中，按如下方式重新编译：

$ gcc -L. -larith main.c -Wl,-rpath='.' -o main
-rpath是链接器选项，并不是gcc的编译选项，所以上面通过-Wl,告知编译器将此选项传给下一阶段的链接器。重新编译后，采用readelf命令查看main文件的dynamic节，发现多了一个RPATH字段，且值就是我们前面设置的路径。

$ readelf -d main| grep PATH
  0x000000000000000f (RPATH)              Library rpath: [.]
再次尝试运行main文件会发现一切正常。

$ORIGIN
上面的解决办法还有一些小问题，RPATH指定的路径是相当于当前目录的，而不是相对于可执行文件所在的目录，那么当换一个目录再执行上面的程序，就会又报找不到共享库。解决这个问题的办法就是使用$ORIGIN变量，在运行的时候，链接器会将该变量的值用可执行文件所在的目录来替换，这样我们就又能相对于可执行文件来指定RPATH了。重新编译如下：
$ gcc -L. -larith main.c -Wl,-rpath='$ORIGIN/' -o main
```



## 相关路径

/lib：最关键和基础的动态链接库。

/usr/lib：关键的动态链接库。

/usr/local/lib：第三方动态链接库。

由/etc/ld.so.conf配置文件指定的目录。



## ldconfig

动态链接器不可能在每次查找动态链接库都去遍历所有动态链接库的目录，这样速度太慢了。因此，在系统启动时会通过ldconfig为动态链接库生成SO_NAME和/etc/ld.so.cache存放系统动态链接库的路径信息，加速动态链接库的查找。

程序启动查找动态链接库的路径顺序如下：

由LD_LIBRARY_PATH指定的路径。

由路径缓存文件/etc/ld.so.cache指定的路径。

默认共享库目录，先/usr/lib，然后/lib。

注意，安装动态链接库后，需要重启系统或运行ldconfig生成SO_NAME和刷新/etc/ld.so.cache文件。



## ldd

通过ldd elf_file可以查看ELF文件依赖哪些动态链接库，如

ldd a

```
        linux-vdso.so.1 =>  (0x00007fffc5bfe000)
    
        libmylib.so => not found
    
        libstdc++.so.6 => /lib64/libstdc++.so.6 (0x00007f258d42d000)
    
        libm.so.6 => /lib64/libm.so.6 (0x00007f258d12b000)
    
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f258cf15000)
    
        libc.so.6 => /lib64/libc.so.6 (0x00007f258cb47000)
    
        /lib64/ld-linux-x86-64.so.2 (0x00007f258d734000)

linux-vdso.so.1是内核提供的一个动态链接库，所以这里只有一个内存地址。

/lib64/ld-linux-x86-64.so.2是一个动态链接库的绝对路径。
```