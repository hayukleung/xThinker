---
title: 使用autotools制作makefile
date: 2012-05-05 22:24:25
tags: c
---
[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/05/05/2485099.html)

## flat模式
- ①#autoscan
```
生成configure.scan
```
- ②改写configure.scan
```
AC_INIT(最终可执行文件名, 版本号)
AM_INIT_AUTOMAKE
AC_CONFIG_SRCDIR(源文件所在文件夹中的一个文件名，用于检测路径)
AC_CONFIG_HEADER(config.h)
AC_OUTPUT(Makefile)
AC_PROG_RANLIB
改写完毕后另存为configure.in
```
- ③#aclocal
- ④#autoconf
- ⑤#autoheader
- ⑥编辑Makefile.am
```
AUTOMAKE_OPTIONS=foreign
bin_PROGRAMS=可执行文件名1可执行文件名2 ......可执行文件名n
可执行文件名1_SOURCES=该可执行文件生成所必须的源文件、头文件等，以空格分开
可执行文件名2_SOURCES=该可执行文件生成所必须的源文件、头文件等，以空格分开
......
可执行文件名n_SOURCES=该可执行文件生成所必须的源文件、头文件等，以空格分开
可执行文件名1_LDADD=-l库名
......
可执行文件名n_LDADD=-l库名
```
- ⑦#automake --add-missing
- ⑧#./configure
- ⑨#make
- ⑩生成可执行文件，运行。

## deep模式
```
以例子说明
环境：Linux Redhat Enterprise
文件结构：
根目录：
/rtcp
子目录：
/rtcp/src：包含server.c client.c
/rtcp/include：包含rtcp.c rtcp.h
目标：在根目录/rtcp下建立Makefile
步骤：
```
- ①进入/rtcp
```
#autoscan
产生configure.scan
修改该文件：
AM_INIT_AUTOMAKE
# Checks for programs.
AC_PROG_CC 
AM_PROG_CC_C_O
# Checks for libraries. 
AC_PROG_RANLIB
AC_OUTPUT(Makefile)
保存为configure.in
```
- ②
```
#aclocal
#autoconf
#autoheader
#gedit Makefile.am
```
- ③编辑Makefile.am
```
AUTOMAKE_OPTIONS=subdir-objects 
bin_PROGRAMS=server client 
server_SOURCES=src/server.c include/rtcp.c include/rtcp.h 
client_SOURCES=src/client.c include/rtcp.c include/rtcp.h 
server_LDADD=-lpthread 
client_LDADD=-lpthread
server_LDFLAGS=-L/path/to/lib/pthread/
client_LDFLAGS=-L/path/to/lib/pthread/
保存
```
- ④#automake --add-missing
- ⑤#./configure
- ⑥#make