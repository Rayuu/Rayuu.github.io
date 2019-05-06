---
title: gcc编译流程解析
date: 2017-03-14 22:45:24
tags: c/c++
---

## gcc编译流程解析


[TOC]


<!--more-->

### 1、预处理

```C
	#include<stdio.h>
	int main()
	{
		printf("Hello! This is our embedded world!\n");
		return 0;
	}
```

在该阶段，对包含的头文件（`#include`）和宏定义（`#define`、`#ifdef`等）进行处理。

可以通过`gcc -E `选项进行查看。该选项的作用是让gcc 在预处理结束后停止编译过程。

`gcc -E hello.c -o hello.i`

`-o `是指目标文件。 `.i`文件为经预处理后的C程序。


### 2、编译阶段


`gcc -S hello.i -o hello.s`
上述命令只进行编译不进行汇编，结果生成汇编代码。

在编译阶段，gcc首先要检查代码的规范性、是否有语法错误。


### 3、汇编阶段


汇编阶段把编译阶段生成的.s文件转成目标文件。

`gcc -c hello.s -o hello.o`

把汇编代码转为二进制目标代码。


### 4、链接阶段

链接时gcc会到系统默认的搜索路径` /usr/lib`下进行查找动态库。

函数库用动态库和静态库两种。

静态库是指编译链接时，将库文件的代码全部加入到可执行文件中。生成文件比较大。在运行时也就不再需要库文件了。后缀名通常为`.a`

动态库与之相反，在编译链接时并没有将库文件的代码加入可执行文件中，而是在程序执行时加载库。这样可以减轻系统的开销。一般动态库的后缀名为`.so` .

> gcc编译时默认使用动态库。

完成了链接之后，gcc就可以生成可执行文件。

`gcc hello.o -o hello`

`./hello`

> gcc编译选项分析

### gcc 常用选项列表

|选 项  |  含 义|
|-------|-------|
| `-c` | 只编译不链接，生成目标文件“ `.o`” 
|`-S`|只编译不汇编，生成汇编代码
|`-E`|只进行预编译，不做其他处理
|`-g`|在可执行程序中包含标准调试信息
|`-o file`|将 file 文件指定为输出文件
|`-v`|打印出编译器内部编译各过程的命令行信息和编译器的版本
|`-I dir`|在头文件的搜索路径列表中添加 dir 目录
`-I dir `选项可以在头问价的搜索路径列表中添加dir目录。

Linux中默认头文件都放到了“`/usr/include/`”目录下。因此如果用户希望添加放置在其他位置的头文件时，就可以通过“-I dir”选项来指定。

`gcc hello.c -I /home/rayu/cstudy/  -o hello`


###　gcc 库选项列表

|选 项|含 义|
|-----|-----|
|-static|进行静态编译，即链接静态库，禁止使用动态库
|-shared|1．可以生成动态库文件 <br /> 2．进行动态编译，尽可能地链接动态库，只有当没有动态库时才会链接同名的静态库（默认选项，即可省略）
|-L dir|在库文件的搜索路径列表中添加 dir 目录-lname链接称为 libname.a（静态库）或者 libname.so（动态库）的库文件。若两个库都存在，则根据编译方式（ -static 还是-shared）而进行链接|
|-fPIC(或-fpic)      |         生成使用相对地址的位置无关的目标代码（ Position Independent Code）。然后通常使用gcc 的-static 选项从该 PIC 目标文件生成动态库文件

### 静态库的创建和使用

```C
/* unsgn_pow.c：库程序 */
unsigned long long unsgn_pow(unsigned int x, unsigned int y)
{
    unsigned long long res = 1;
    if (y == 0)
    {
        res = 1;
    }
    else if (y == 1)
    {
        res = x;
    }
    else
    {
        res = x * unsgn_pow(x, y - 1);
    }
    return res;
}
```
```C
#include<stdio.h>
#include<stdlib.h>

int main(int argc,char *argv[])
{
    unsigned int x,y;
    unsigned long long res;
    if((argc<3) || (sscanf(argv[1],"%u",&x)!=1) || (sscanf(argv[2],"%u",&y))!=1)
    {
        printf("Usage: pow base exponent\n");
        exit(1);
    }
    res = unsgn_pow(x,y);
    printf("%u ^ %u = %u\n",x,y,res);
    exit(0);
}
```

```
gcc -c unsgn_pow.c
ar rcsv libpow.a unsgn_pow.o
gcc -o pow_test pow_test.c -L. -lpow
./pow_test 2 10
```

### 动态库的创建和使用

首先使用 `gcc  -fPIC`选项为动态库构造一个目标文件

`gcc -fPIC -Wall -c unsgn_pow.c`

接下来，使用`-shared`选项和已创建的位置无关目标代码，生成一个动态库`libpow.so`

`gcc -shared -o libpow.so unsgn_pow.o`

下面编译主程序，它将会链接到刚刚生成的动态库`libpow.so`

`gcc -o pow_test pow_test.c -L. -lpow`

在运行可执行程序之前，需要注册动态库的路径名。其方法有几种：修改`/etc/ld.so.conf`文件，或者修改`LD_LIBRARY_PATH`环境变量，或者将库文件直接复制到`/lib`或者`/usr/lib`目录下（这两个目录为同的默认的库路径名）

```
cp libpow.so /lib
./pow_test 2 10
```

### 告警和出错选项

gcc 的告警和出错选项。

gcc 警告和出错选项选项列表

|选 项|含 义|
|-----|-----|
|-ansi|支持符合 ANSI 标准的 C 程序
|-pedantic|允许发出 ANSI C 标准所列的全部警告信息
|-pedantic -error|允许发出 ANSI C 标准所列的全部错误信息
|-w|关闭所有告警
|-Wall |                   允许发出 gcc 提供的所有有用的报警信息
|-Werror     |          把所有的告警信息转化为错误信息，并在告警发生时终止编译过程

`gcc –Wall  –O -g –c kang.c -o kang.o`

`-Wall` 是打开警告开关，
`-O`代表默认优化，可选：-O0不优化，-O1低级优化，-O2中级优化，-O3高级优化，-Os代码空间优化。
`-g`是生成调试信息，生成的可执行文件具有和源代码关联的可调试的信息。



