---
layout: post
title: "静态链接"
categories: 
tags: 
permalink: /12/
---

# 静态链接

首先回顾从 C 源代码编译获得可执行文件的过程：
1. 编译。编译又可以细分为几个阶段，产物是汇编代码。
    1. 预编译。
    2. 词法分析。
    3. 语法分析。
    4. 语义分析。
    5. 中间语言生成。
    6. 目标代码生成和优化。
2. 汇编。从汇编代码生成机器代码和其他链接所需信息，产物是可重定位文件。
3. 链接。将多个目标文件合并，处理目标文件之间的符号引用关系，生成可执行文件。
    1. 空间与地址分配。
    2. 符号决议。
    3. 重定位。

前一篇文章介绍了汇编这个步骤的产物——可重定位文件的各个部分的内容。虽然可重定位文件已经是二进制机器代码，但仍不能正确地被机器执行。这是因为代码中引用了其他模块中的函数（例如标准库中的 `printf` 函数），这些定义在其他地方的函数（或变量）对本目标文件不可知，也就不能执行这些函数，只在引用处留了一个标记，等待链接时重新定位这些函数。此外，可重定位文件没有可执行文件的结构，不能被装载入内存。

前一篇文章中还简要介绍了重定位的过程，这也是链接中的重要步骤，但链接要完成的事情远不止符号重定位。链接分为静态链接和动态链接两种，静态链接生成的可执行文件中包含了程序运行所需的所有内容，而动态链接会在运行时动态地载入程序所使用的函数库（共享目标文件）。静态链接文件在运行不依赖其他文件（不考虑运行时读取文件的情况），因而其过程相对动态链接来说更为简单。本文就以较为简单的静态链接为切入点尝试介绍链接的主要过程。

## 空间与地址分配

下面的示例代码中，`swap.c` 提供了一个全局变量和一个函数 `swap`，这个函数通过位运算“异或”交换两个数的值。`main.c` 则调用 `swap` 函数，将一个局部变量的值与全局变量交换。

```c
/* main.c*/
void swap(long*, long*);
extern long shared;
int main() {
  long a = 100;
  swap(&a, &shared);
}
```

```c
/* swap.c */
long shared = 1;
void swap(long* a, long* b) { 
  *a ^= *b ^= *a ^= *b; 
}
```

通过 `gcc -static main.c swap.c -O1` 可以将两个模块静态链接成可执行文件 `a.out`。`-static` 参数表示静态链接，如果不加这个参数 GCC 默认会执行动态链接。也可以先执行 `gcc -c main.c swap.c -O1` 得到两个源代码文件对应的可重定位文件 `main.o` 和 `swap.o`，再通过 `gcc -static main.o swap.o` 得到可执行文件 `a.out`。

回顾可重定位文件的段表，其中有一列 `Address` 表示段的虚拟内存地址。可重定位文件中这个值都是 0，因为可重定位文件不能被装载（Loading），因而也就不会有虚拟内存地址。静态链接的输入是若干个可重定位文件，输出是一个可执行文件。可执行文件可以被装载入内存，因此可执行文件中的段会有一个有效的虚拟内存地址。

借助 `readelf` 查看可执行文件的段信息。这里只展示部分段表信息。
```sh
$ readelf -S a.out
There are 29 section headers, starting at offset 0xcab10:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
...
  [ 6] .text             PROGBITS         00000000004010c0  000010c0
       000000000007b13d  0000000000000000  AX       0     0     64
  [22] .data             PROGBITS         00000000004b40a0  000b30a0
       00000000000019e8  0000000000000000  WA       0     0     32
  [23] .bss              NOBITS           00000000004b5aa0  000b4a88
       0000000000005828  0000000000000000  WA       0     0     32       

```

可重定位文件中包含了各个类型的段，经过静态链接，来自不同可重定位文件中的相同类型段合并为了一个段。观察 `.text` 段的属性可以发现，该段的虚拟内存地址为 `0x004010c0`，同时该段在可执行文件中的偏移是 `0x000010c0`。不难发现，虚拟内存地址比文件内的偏移多了 `0x00400000`，这个值也是一般的 64 位 ELF 可执行文件载入内存后，进程虚拟内存空间的起始地址。

在链接过程中，链接器扫描各个输入文件中段的信息，合并相同类型段后可以计算这些段的总长度，进而确定空间分布并分配有效的虚拟内存地址。这一步完成后，代码和符号就有了唯一确定的运行时虚拟内存地址。

## 符号决议 Symbol Resolution

回顾可重定位文件，代码段中唯一不确定的就是符号的地址。经过上一步的空间和地址分配，链接需要的所有可重定位文件中的所有符号就合并到了一个符号表中，并且这些符号也被分配了虚拟内存地址。同时，各个段的可重定位段也合并，在符号地址确定后，就可以根据重定位表的信息修改重定位入口的符号值，这是也链接最主要的任务。

不过在真正开始重定位前，还有一个问题要解决，那就是符号名称冲突。在合并完符号表后，如果发现有符号的名称相同，这意味着不同的可重定位文件中存在名称相同的全局变量或函数。如果在单个模块内的局部变量如果使用了相同的名称，编译器在编译阶段就会给出重定义错误。局部变量只对模块内部可见，不参与链接，而全局变量和函数在链接时对其他模块可见，如果几个模块中出现全局变量或者函数重名的情况，链接器应当如何选择？链接器需要有一个规则来确定如何在多个重名符号之中选择一个，或者给出错误提示，停止链接。

考虑这样的情况，将每个模块分布写在单独的 C 文件中，再将这些模块各自编译成可重定位文件。当新模块需要使用某个模块的功能时，只要使用这些模块中对应的符号名就能通过编译得到可重定位文件。
当使用某个模块中的全局变量时，通过 C 关键字 `extern` 在本模块中声明；当使用某个模块中的函数时，需要先在本模块中引入函数声明或包含函数声明的头文件，再调用函数。在链接时，再将使用到的各个可重定位文件作为链接器的参数，由链接器得到可执行文件。一个模块中可以引用定义在其他模块中的符号，确定符号引用的工作由链接器完成。链接器在确定符号引用时，只会将符号引用指向一个唯一的符号定义，如果出现冲突，链接需要确定使用哪一个符号或者给出错误，这个过程称为符号决议。

为了避免符号重名的问题，在编译阶段，编译器将模块中所有全局符号分为强符号和弱符号，并将信息导出给汇编器。汇编器将这些符号信息一并写入可重定位文件的符号表。**函数**和**初始化的全局变量**是**强符号**（Strong symbol），**未初始化的全局变量**是**弱符号**（Weak symbol）。有了符号分类后，链接器遵循下面三条规则来处理名称冲突的符号。
1. 如果强符号名称冲突，那么链接至此终止。
2. 如果一个强符号和多个弱符号的名称冲突，那么选择强符号。
3. 如果多个弱符号名称冲突，那么选择任意一个弱符号。

如果两个模块中出现在同名强符号，依据规则 1，链接器就会给出重定义的错误提示，形如 `multiple definition of ``foo'; foo.o:foo.c:(.text+0x0): first defined here`。而当一个模块中存在一个初始化的全局变量，另一个模块中存在同名但没有初始化的全局变量时，依据规则 2，链接器会**隐式**地选择已初始化的强符号作为这个符号的定义。这意味着，其他所有使用了这个符号名称的地方都会将引用指向链接器选择的符号，即便在模块代码中可能并不是要使用这个符号。这虽然能完成链接，但在一些地方可能会使代码不符合预期。相同的问题也会发生在应用规则 3 的情况下，甚至更糟糕。例如当一处定义的变量为 `int`，另一处为 `double` 时，如果另一处在代码中给变量赋一个 `double` 值，由于 `double` 为 8 字节，赋值时可能会意外地写入以 `int` 作为变量类型的内存区域的相邻的 4 个字节区域。

为了避免这类问题，在编译时可以启用 `-fno-common` 选项。在较早的描述符号决议的文章中，经常会看到有关 COMMON 块的表述。COMMON 块是可重定位文件中用于存放未初始化的全局变量的段，也就是弱符号存放的段。如果全局变量没有赋值就会存放于 COMMON 块当中等待链接器确定符号，而如果将变量初始化为 0，这个变量就会被存放在 `.bss` 段中，变成强符号。强符号在链接时如果出现名称冲突，链接器一定会输出错误，因而能够即使发现并修改代码，避免任何链接器隐式选择造成的不可预料结果。而 `-fno-common` 选项可以编译器自动给未赋值的全局变量赋值为 0，从而避免使用 COMMON 块。

幸运的是，从 GCC 10.1 起[^1][^2]， `-fno-common` 选项就成为了默认选项，不必再担忧这个问题，未初始化和初始化为 0 的全局变量都会被当作强符号。


## 重定位 Relocation

解决了符号决议的问题，就可以放心地来给符号重定位了。

### 验证符号重定位

在介绍可重定位文件时，提到过 `R_X86_64_PC32` 这种重定位类型，这里来验证链接后可执行文件中重定位地址的变化。将前面的两个示例代码文件编译 `gcc -c main.c swap.c -O1`，得到可重定位文件 `main.o` `swap.o`。检查 `main.o` 文件中的重定位信息，执行 `objdump -r -d main.o`，有如下输出。

```sh
$ objdump -r -d main.o 

main.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <main>:
   0:   48 83 ec 18             sub    $0x18,%rsp
   4:   64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
   b:   00 00 
   d:   48 89 44 24 08          mov    %rax,0x8(%rsp)
  12:   31 c0                   xor    %eax,%eax
  14:   48 c7 04 24 64 00 00    movq   $0x64,(%rsp)
  1b:   00 
  1c:   48 89 e7                mov    %rsp,%rdi
  1f:   48 8d 35 00 00 00 00    lea    0x0(%rip),%rsi        # 26 <main+0x26>
                        22: R_X86_64_PC32       shared-0x4
  26:   e8 00 00 00 00          call   2b <main+0x2b>
                        27: R_X86_64_PLT32      swap-0x4
  2b:   48 8b 44 24 08          mov    0x8(%rsp),%rax
  30:   64 48 2b 04 25 28 00    sub    %fs:0x28,%rax
  37:   00 00 
  39:   75 0a                   jne    45 <main+0x45>
  3b:   b8 00 00 00 00          mov    $0x0,%eax
  40:   48 83 c4 18             add    $0x18,%rsp
  44:   c3                      ret
  45:   e8 00 00 00 00          call   4a <main+0x4a>
                        46: R_X86_64_PLT32      __stack_chk_fail-0x4
```

反汇编结果中的第 22 字节是一个 `R_X86_64_PC32` 类型的重定位入口，对应源代码中使用共享全局变量 `shared` 的位置。在可重定位文件中，其值为 0。

将两个可重定位文件链接 `gcc -static main.o swap.o -o a.out`，再执行反汇编查看可执行文件中 `main` 函数的反汇编结果。

```sh
$ objdump -d a.out | grep "<main>" -A33
0000000000402e85 <main>:
  402e85:       48 83 ec 18             sub    $0x18,%rsp
  402e89:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
  402e90:       00 00 
  402e92:       48 89 44 24 08          mov    %rax,0x8(%rsp)
  402e97:       31 c0                   xor    %eax,%eax
  402e99:       48 c7 04 24 64 00 00    movq   $0x64,(%rsp)
  402ea0:       00 
  402ea1:       48 89 e7                mov    %rsp,%rdi
  402ea4:       48 8d 35 05 12 0b 00    lea    0xb1205(%rip),%rsi        # 4b40b0 <shared>
  402eab:       e8 1f 00 00 00          call   402ecf <swap>
  402eb0:       48 8b 44 24 08          mov    0x8(%rsp),%rax
  402eb5:       64 48 2b 04 25 28 00    sub    %fs:0x28,%rax
  402ebc:       00 00 
  402ebe:       75 0a                   jne    402eca <main+0x45>
  402ec0:       b8 00 00 00 00          mov    $0x0,%eax
  402ec5:       48 83 c4 18             add    $0x18,%rsp
  402ec9:       c3                      ret
  402eca:       e8 b1 0f 01 00          call   413e80 <__stack_chk_fail>

0000000000402ecf <swap>:
  402ecf:       48 8b 07                mov    (%rdi),%rax
  402ed2:       48 33 06                xor    (%rsi),%rax
  402ed5:       48 89 07                mov    %rax,(%rdi)
  402ed8:       48 33 06                xor    (%rsi),%rax
  402edb:       48 89 06                mov    %rax,(%rsi)
  402ede:       48 31 07                xor    %rax,(%rdi)
  402ee1:       c3                      ret
  402ee2:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
  402ee9:       00 00 00 
  402eec:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
  402ef3:       00 00 00 
  402ef6:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
  402efd:       00 00 00 
```

反汇编结果中，对 `shared` 全局变量的引用指令地址已经被修正为了 `0xb1205`。计算 `0x402eab` + `0xb1205` 得到变量 `shared` 的地址为 `0x4b40b0`，这与注释吻合。

检查可执行文件的数据段，`readelf -j .data a.out`。可以发现在地址 `0x004b40b0` 开始的 8 字节中存放的正是 `shared` 的值 1。

```sh
$ readelf -j .data a.out | head

Hex dump of section '.data':
  0x004b40a0 00000000 00000000 00000000 00000000 ................
  0x004b40b0 01000000 00000000 00080000 00000000 ................
  0x004b40c0 00080000 00000000 00001000 00000000 ................
  0x004b40d0 00000800 00000000 00800000 00000000 ................
  0x004b40e0 00400000 00000000 c0634b00 00000000 .@.......cK.....
  0x004b40f0 01000000 00000000 00000000 00000000 ................
  0x004b4100 ffffffff 00000000 01000000 00000000 ................
  0x004b4110 00000000 00000000 00000000 00000000 ................
```

### R_X86_64_PLT32

重定位类型 `R_X86_64_PLT32` 是基于 PLT（Procedure Linkage Table）的来计算符号位置，而 `R_X86_64_PC32` 是基于下一条指令的地址来计算符号位置。PLT 是在动态链接中应用的延迟绑定机制，使得动态链接程序不必在程序载入时将所有引用的符号完成重定位，而是在每个符号真正调用时再完成重定位，这种机制优化了程序性能。而在静态链接程序中，由于没有使用共享库，函数定义在同一个文件中，重定位类型 `R_X86_64_PLT32` 会被当作 `R_X86_64_PC32` 一样处理。反汇编代码中给出了 `swap` 这个符号的位置，依据 `R_X86_64_PC32` 重定位类型的地址修正方式可以计算这个位置修正后的值。这里不再计算，只做验证。观察反汇编代码中对 `swap` 函数的调用，重定位符号值为 `0x1f`,下一条指令地址位 `0x402eb0`,两者之和是 `0x402ecf`，与下方 `swap` 函数的位置匹配。


## 静态链接实践

理解了链接的三个主要部分之后，再来看实践中的应用。

在前面的例子中，通过 `gcc -static main.o swap.o -o a.out` 将目标文件链接为可执行文件。GCC 的输入文件列表是两个可重定位文件，链接后两个文件的内容合并。实际上，在链接阶段，链接器的输入不止这两个文件。一般的 C 程序通过直接或间接地使用到标准 C 库的函数和功能，常见的标准 C 库是 GLIBC。例如，如果用到了 `printf` 在链接过程中就会将 `printf.o` 也加入链接器的输入文件列表；`scanf` 则对应 `scanf.o`。每个库函数都对应一个可重定位文件，这可以使链接后输出的可执行文件只包含必要的代码。一般情况下，输入给链接器的所有 `.o` 文件中的函数变量都会合并到最终的可执行文件中，为了减少可执行文件体积，C 库中的函数编写在单独的文件中，链接时只选择需要的 `.o` 文件以使可执行文件尽可能小。

### 静态库

一般程序通常会用到数量众多的标准库函数，如果链接时要将用到的库函数对应 `.o` 文件依此作为参数传递给链接器，那么链接器接收到的参数量将会相当大。为了简化这个过程，方便函数库的管理、传输，多个 `.o` 文件会使用 `ar` 程序压缩为单个文件，形成静态库文件。

#### 制作静态库

下面有两份源代码，分别存为 `addvec.c` 和 `multvec.c`。

```c
int addcnt = 0;

void addvec(int* x, int* y, int* z, int n) {
  int i;
  addcnt++;

  for (i = 0; i < n; i++) {
    z[i] = x[i] + y[i];
  }
}
```

```c
int multcnt = 0;

void multvec(int* x, int* y, int* z, int n) {
  int i;
  multcnt++;
  
  for (i = 0; i < n; i++) {
    z[i] = x[i] * y[i];
  }
}
```

通过 `gcc -c addvec.c multvec.c -O1` 编译，获得 `addvec.o` 和 `multvec.o` 两个可重定位文件。再通过 `ar rcs libvector.a addvec.o multvec.o` 将两个可重定位文件合并，获得静态库文件 `libvector.a`。通过 `ar t libvector.a` 可以查看包含在静态库中的可重定位文件列表。通过 `objdump -t libvector.a` 可以查看每个可重定位文件的符号表。通过 `ar -x libvector.a` 可以将静态库中的所有可重定位文件解压出来。

GLIBC 提供的静态库文件是 `libc.a`，其中包含了超过两千个 C 函数。前面的编译命令中虽然没有显式的将 `libc.a` 作为参数，但 GCC 在识别到 `-static` 静态链接选项后会自动将 `libc.a` 作为链接时的输入参数。`libm.a` 是与数学有关的静态库文件，三角函数、指数、对数等数学函数就包含在这个静态库中，数量接近一千个。

### 静态库链接
有了静态库，编译时只需要给出静态库文件的位置，链接阶段就能自动从中提取出需要引用符号所在的可重定位文件加入链接。

将下面的代码存为 `main2.c`，与静态库文件 `libvector.a` 静态链接。

```c
void addvec(int* x, int* y, int* z, int n);
int printf(const char *format, ...);

int x[2] = {1, 2};
int y[2] = {3, 4};
int z[2];

int main() {
  addvec(x, y, z, 2);
  printf("z= [%d,%d]\n", z[0], z[1]);
  return 0;
}
```

先通过 `gcc -c main2.c -O1` 获得 `main2.o` 后，再通过 `gcc -static main2.o libvector.a -o vector.out` 获得可执行文件 `vector.out`。

上面的代码中，引用了 `addvec` 符号，该符号存在于静态库 `libvector.a` 中，链接器可从该静态库中检索到所在可重定位文件 `addvec.o`。而 `printf` 符号所在的 `printf.o` 包含在 `libc.a` 中，该静态库由 GCC 自动添加。

#### 静态库的符号检索

在符号决议阶段，链接器会按照输入参数从左往右扫描输入的文件，包括可重定位文件和静态库文件。如果是直接从源代码文件编译得到可执行文件，编译器会先将源代码编译成可重定位文件再链接。在这个扫描过程中，链接器维护一个可重定位文件的集合 $$E$$，这个集合中的文件最终会合并到可执行文件中。集合 $$U$$ 存放所有尚未完成找到定义的符号，集合 $$D$$ 存放已经找到定义的符号。初始阶段这三个集合为空。

1. 对于每个输入文件 $$f$$，链接器先判断其是目标文件还是静态库文件。如果是目标文件，那么该文件加入集合 $$E$$，同时更新另外两个集合以反映当前输入文件中的符号定义和引用。然后继续处理下一个文件。
2. 如果输入文件是静态库，链接器会尝试在静态库的符号列表中寻找集合 $$U$$ 中的符号。如果找到了定义了包含在集合 $$U$$ 中的若干符号的目标文件 $$m$$，那么将 $$m$$ 加入集合 $$E$$，同时更新 $$U$$ 和 $$D$$。这个过程会遍历完静态库文件中的所有目标文件，直到集合不再变化后，静态库文件中没有加入到集合 $$E$$ 的目标文件将被丢弃。可以理解为这个过程从静态库文件中提取需要的目标文件，提取完成后这个静态库文件后续将不再使用。
3. 当链接器处理完所有输入文件后，如果集合 $$U$$ 不为空，说明仍有符号没有找到定义，那么链接器输出错误并终止。如果所有符号都找到了定义，那就可以继续构建可执行文件。

这样的符号检索规则需要精心设计链接器的输入文件次序。例如，如果前面的静态链接命令修改为 `gcc -static libvector.a main2.o -o vector.out`，那链接器就找不到 `main2.o` 中使用的 `addvec` 符号，因为其所在的静态库位于第一个参数，当其被链接器处理时，`addvec` 参数尚未进入集合 $$U$$，因而整个静态库被丢弃，后续就找不到符号定义。

为了能够正确链接，一般将静态库放在链接器参数的末尾。如果静态库之间存在依赖，要将更为独立的静态库放在更后面。如果静态库之间存在循环依赖，可以将同一个静态库重复放在链接器参数中。链接器 `ld` 也提供了 `--start-group` 和 `--end-group` 参数，在这两个参数之间的静态库会在解决了循环依赖之后再丢弃。

## 静态链接的最后部分

通过前面的介绍可以理解链接的基本原理，也可以借此分析一些实践中的问题。这里介绍完成链接的最后部分，这部分让链接后的文件成为可执行文件。

### GCC 隐藏的链接信息

注意到，前面给出的链接指令都是通过 GCC 来完成的，而 GCC 并不是链接器，它只是将参数传递给了链接器。为了观察 GCC 执行的完成过程，给 GCC 加入 `--verbose` 参数。

下面是 `gcc -static main2.c addvec.c -O1 -o vector.out -fno-builtin --verbose` 的输出。
```sh
$ gcc -static main2.c addvec.c -O1 -o vector.out -fno-builtin --verbose
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/lto-wrapper
Target: x86_64-pc-linux-gnu
Configured with: /build/gcc/src/gcc/configure --enable-languages=ada,c,c++,d,fortran,go,lto,m2,objc,obj-c++,rust,cobol --enable-bootstrap --prefix=/usr --libdir=/usr/lib --libexecdir=/usr/lib --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=https://gitlab.archlinux.org/archlinux/packaging/packages/gcc/-/issues --with-build-config=bootstrap-lto --with-linker-hash-style=gnu --with-system-zlib --enable-__cxa_atexit --enable-cet=auto --enable-checking=release --enable-clocale=gnu --enable-default-pie --enable-default-ssp --enable-gnu-indirect-function --enable-gnu-unique-object --enable-libstdcxx-backtrace --enable-link-serialization=1 --enable-linker-build-id --enable-lto --enable-multilib --enable-plugin --enable-shared --enable-threads=posix --disable-libssp --disable-libstdcxx-pch --disable-werror
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 15.2.1 20250813 (GCC) 
COLLECT_GCC_OPTIONS='-static' '-O1' '-o' 'vector.out' '-fno-builtin' '-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'vector.out-'
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/cc1 -quiet -v main2.c -quiet -dumpdir vector.out- -dumpbase main2.c -dumpbase-ext .c -mtune=generic -march=x86-64 -O1 -version -fno-builtin -o /tmp/cc8byjZ0.s
GNU C23 (GCC) version 15.2.1 20250813 (x86_64-pc-linux-gnu)
        compiled by GNU C version 15.2.1 20250813, GMP version 6.3.0, MPFR version 4.2.2, MPC version 1.3.1, isl version isl-0.27-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
ignoring nonexistent directory "/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../x86_64-pc-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/include
 /usr/local/include
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/include-fixed
 /usr/include
End of search list.
Compiler executable checksum: 09af9d42b9185573b6c370dd7aa39e64
COLLECT_GCC_OPTIONS='-static' '-O1' '-o' 'vector.out' '-fno-builtin' '-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'vector.out-'
 as -v --64 -o /tmp/ccjCivlq.o /tmp/cc8byjZ0.s
GNU assembler version 2.45.0 (x86_64-pc-linux-gnu) using BFD version (GNU Binutils) 2.45.0
COLLECT_GCC_OPTIONS='-static' '-O1' '-o' 'vector.out' '-fno-builtin' '-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'vector.out-'
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/cc1 -quiet -v addvec.c -quiet -dumpdir vector.out- -dumpbase addvec.c -dumpbase-ext .c -mtune=generic -march=x86-64 -O1 -version -fno-builtin -o /tmp/cc8byjZ0.s
GNU C23 (GCC) version 15.2.1 20250813 (x86_64-pc-linux-gnu)
        compiled by GNU C version 15.2.1 20250813, GMP version 6.3.0, MPFR version 4.2.2, MPC version 1.3.1, isl version isl-0.27-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
ignoring nonexistent directory "/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../x86_64-pc-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/include
 /usr/local/include
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/include-fixed
 /usr/include
End of search list.
Compiler executable checksum: 09af9d42b9185573b6c370dd7aa39e64
COLLECT_GCC_OPTIONS='-static' '-O1' '-o' 'vector.out' '-fno-builtin' '-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'vector.out-'
 as -v --64 -o /tmp/ccJw1gTQ.o /tmp/cc8byjZ0.s
GNU assembler version 2.45.0 (x86_64-pc-linux-gnu) using BFD version (GNU Binutils) 2.45.0
COMPILER_PATH=/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/:/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/:/usr/lib/gcc/x86_64-pc-linux-gnu/:/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/:/usr/lib/gcc/x86_64-pc-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/:/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../lib/:/lib/../lib/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-static' '-O1' '-o' 'vector.out' '-fno-builtin' '-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'vector.out.'
 /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/collect2 -plugin /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/lto-wrapper -plugin-opt=-fresolution=/tmp/cccIKp6Z.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_eh -plugin-opt=-pass-through=-lc --build-id --hash-style=gnu -m elf_x86_64 -static -o vector.out /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../lib/crt1.o /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../lib/crti.o /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/crtbeginT.o -L/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1 -L/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../lib -L/lib/../lib -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../.. -L/lib -L/usr/lib /tmp/ccjCivlq.o /tmp/ccJw1gTQ.o --start-group -lgcc -lgcc_eh -lc --end-group /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/crtend.o /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/../../../../lib/crtn.o
COLLECT_GCC_OPTIONS='-static' '-O1' '-o' 'vector.out' '-fno-builtin' '-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'vector.out.'
```

命令将源代码直接编译成了可执行文件，通过输出可以观察到编译每一步执行的过程。参数 `-fno-builtin` 防止 GCC 使用内建函数优化源代码中的函数。
- `cc1` 是 GCC 的 C 语言编译器，它将源代码编译得到临时文件 `/tmp/ccav0mO3.s`。这是一个汇编语言文件。
- `as -v --64 -o /tmp/ccPrnJ1f.o /tmp/ccav0mO3.s`，这条指令调用汇编器 `as`，将上面的汇编语言文件汇编得到目标文件 `/tmp/ccPrnJ1f.o`，这也是一个临时文件。
- 最后调用 `collect2` 完成链接，得到可执行文件 `vector.out`。

`collect2` 是链接器 `ld` 的包装，用于处理 C++ 中的构造函数。


完成了链接的三个部分就足够创建一个可执行文件了吗？
链接了什么？
C 语言运行库。堆栈初始化，入口函数，初始化，资源清理函数。
GCC 库的部分。




重定位（Relocation）

静态链接

`ld -static /usr/lib/crt1.o /usr/lib/crti.o /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/crtbeginT.o -L /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/ -L /usr/lib/ *.o --start-group -lgcc -lgcc_eh -lc --end-group /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/crtend.o /usr/lib/crtn.o `

```sh
ld -static /usr/lib/crt1.o /usr/lib/crti.o -L /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1 
*.o 
--start-group -lgcc -lgcc_eh -lc 
--end-group /usr/lib/crtn.o -verbose
```
group 内部存在循环依赖
信号处理、环境变量、语言区域设定、堆栈管理
lgcc 数值计算指令、异常处理
运行库部分，gcc 库部分。
CRT 通用运行时

ld 链接器脚本



可执行文件（Executable Object Files）

疑问：为什么一个简单的 hello world 代码经过编译、动态链接后会变得那么大？静态链接甚至更大？
ld 参数中的 .o 目标文件会被全部合并到可执行文件中，即使其中的函数、变量没有在其他地方使用。
静态共享库 .a，链接时只会提取代码中用到的函数、变量，动态共享库也是如此，但不会直接将用到的部分合并入可执行文件，而是在程序运行时链接。

如何创建一个只有必要代码的可执行文件？（最小的可执行文件）


装载


## 脚注
[^1]: GCC Bugzilla [-fno-common should be default](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=85678#c15)
[^2]: [GCC 10.1 commit](https://gcc.gnu.org/cgit/gcc/commit/?h=releases/gcc-10.1.0&id=6271dd984d7f920d4fb17ad37af6a1f8e6b796dc)

https://reverseengineering.stackexchange.com/questions/1992/what-is-plt-got
https://gist.github.com/reveng007/b9ef8c7c7ed7a46b10a325f4dee42ac4
https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html

https://reverseengineering.stackexchange.com/questions/2172/why-are-got-and-plt-still-present-in-linux-static-stripped-binaries?lq=1

https://stackoverflow.com/questions/64424692/how-does-the-address-of-r-x86-64-plt32-computed
https://sourceware.org/git/?p=binutils-gdb.git;a=commitdiff;h=bd7ab16b4537788ad53521c45469a1bdae84ad4a;hp=80c96350467f23a54546580b3e2b67a65ec65b66