---
title: FORTRAN编译问题
date: 2022-10-25 09:18:14
toc: true
tags: FORTRAN
categories:
  - model
excerpt: 编译问题Please verify that both the operating system and the processor support Intel(R) X87, CMOV, MMX, FXSAVE, SSE, SSE2, SSE3 and SSSE3 instructions.
---



### make成功，运行时候出现问题（切换到预报服务器和AMD服务器上出现的问题）

- Please verify that both the operating system and the processor support Intel(R) X87, CMOV, MMX, FXSAVE, SSE, SSE2, SSE3 and SSSE3 instructions.

删除FFLAG中的 **-xSSSE3** 选项

```shell
# change from 
FFLAGS = -O3 -ip -march=core2 -mtune=core2 -xSSSE3 -axSSSE3 -fp-model precise -lpthread -qopenmp
# to 
FFLAGS = -O3 -ip -march=core2 -mtune=core2 -axSSSE3 -fp-model precise -lpthread -qopenmp
```



报错问题：程序运行报错，`Fatal Error: This program was not built to run in your system. Please verify that both the operating system and the processor support Intel(R) AVX. yhrun: error: cn2375: task 0: Exited with exit code 1`

原因：该错误说明程序的编译时环境和运行时环境不一致，即程序编译时使用了支持 `AVX` 的选项，运行时的硬件环境不支持该 AVX 优化。一般这种情况发生是由于用户在编译程序时加入 `-xHOST/-xAVX` 选项（或是在安装软件时，系统自动读取到登陆节点上 CPU 的 `flag` 支持 `avx` ，故在编译软件时加入了 `-xHOST`），那程序就会根据登陆节点的 CPU 配置信息进行优化编译，然而程序的运行是在计算节点上，计算节点的 CPU配置信息可能不支持 AVX，就与登陆节点不同，就会报上面的提示错误。

解决方法：编译时去掉 `-xHOST/-xAVX` 选项，使用其他优化选项。

备注：-xHost will cause icc/icpc or icl to check the cpu information and find the highest level of extended instructions support to use.

天河登陆节点 ln1、ln2、ln3 上的 CPU 配置信息 `flag` 均无 `avx` ，ln8、ln9上均有 `avx` 。

如果在 ln8 或 ln9 上安装软件时，configure 后一定要检查下编译 flag 是否加入了 `-xHOST`，如果加入，请修改对应的 configure 文件，将 `-xHOST` 删除。