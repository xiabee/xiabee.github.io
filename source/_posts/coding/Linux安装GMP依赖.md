---
title: Linux安装GMP依赖
date: 2020-9-14 20:00:23
tags:
  - linux
categories:
  - coding
abbrlink: linux-gmp
---



## GMP简介

* [官网](https://gmplib.org/)
* GMP：GNU Multiple Precision Arithmetic Library，即GNU**高精度算术运算库**，在现代密码学的计算中十分关键

> 什么是GMP？
>
> GMP是一个用于任意精度算术的免费库，可处理带符号整数，有理数和浮点数。除了运行GMP机器中的可用内存所暗示的精度外，对精度没有实际限制。GMP具有丰富的功能集，并且这些功能具有常规接口。
>
> GMP的主要目标应用程序是密码学应用程序和研究，Internet安全应用程序，代数系统，计算代数研究等
>
> GMP经过精心设计，无论是小型操作数还是大型操作数，都应尽可能快。通过使用全字作为基本算术类型，使用快速算法，针对许多CPU的最常见内部循环使用经过高度优化的汇编代码并总体上强调速度，可以实现速度。
>
> GMP的第一版发布于1991年。它不断开发和维护，每年大约发布一次新版本。
> 从版本6开始，GMP在GNU LGPL v3 和GNU GPL v2双重许可下分发 。这些许可使该库可以免费使用，共享和改进，并允许您传递结果。GNU许可证赋予了自由，但也对非自由程序的使用设置了严格的限制。
>
> GMP是GNU项目的一部分。有关GNU项目的更多信息，请参见GNU官方网站。
>
> GMP的主要目标平台是Unix类型的系统，例如GNU / Linux，Solaris，HP-UX，Mac OS X / Darwin，BSD，AIX等。它还可以在Windows上以32位和64-位模式。
>
> 摘自GMP官网



安装方式：官网下载，手动安装。这里我们以 **Centos7** 为例



## 下载安装包

进入[官网](https://gmplib.org/)下载最新安装包，这里我们以  `gmp-6.2.0.tar.xz`为例

![img](https://xiabee.cn/wp-content/uploads/2020/09/image-8.png)

然后解压：

```bash
xz -d gmp-6.2.0.tar.xz  
tar xzf gmp-6.2.0.tar 
cd gmp-6.2.0 
```



## 安装依赖

安装GMP 之前需要先安装m4 ，不然会有各种蜜汁报错......不过m4可以直接包管理器安装

```bash
sudo yum install m4
```



## 添加链接

configure一下，同时要加上 --enable-cxx命令，否则不能使用c++库gmpxx.h

```bash
./configure --enable-cxx
```



## 编译运行

```bash
make
make check
sudo make install
```



## 测试

这个时候理论上已经安装好了，我们写一个C语言程序试试：

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <gmp.h>

int main()
{
        mpz_t a,b;

        mpz_init(a);
        mpz_init(b);

        mpz_init_set_ui(a, 2);
        mpz_pow_ui(b, a, 1000);
        gmp_printf("b = %Zd\n", b);

        mpz_clear(a);
        mpz_clear(b);

        return 0;
}
// 计算2的1000次方
```



编译运行：

```bash
gcc -o test test.c -lgmp
# 使用-l命令，寻找gmp链接库

./test
# 执行test文件
```



样例程序输出：

```bash
b = 10715086071862673209484250490600018105614048117055336074437503883703510511249361224931983788156958581275946729175531468251871452856923140435984577574698574803934567774824230985421074605062371141877954182153046474983581941267398767559165543946077062914571196477686542167660429831652624386837205668069376
```



然后就可以去暴算现代密码学了（划掉）