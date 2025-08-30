# 解读目标文件的每一个字节

## 目标文件

目标文件（Object File）是源代码经过编译但尚未链接的中间文件，是包含机器代码的二进制文件。在 Unix 平台上的目标文件遵循 ELF（Executable Linkable Format）文件格式存储。下面四种类型的文件使用 ELF 格式存储：

- 可重定位文件（Relocatable File）。Linux 上的 `.o` 文件，包含代码和数据，可以与其他可重定位文件链接成可执行文件或共享目标文件。静态链接库文件是可重定位文件的归档文件。
- 可执行文件（Executable File）。可以直接在目标平台上运行的文件，也可以称为“程序”，在 Linux 上一般没有扩展名，大家熟知的 `a.out` 就是可执行文件。
- 共享目标文件（Shared Object File）。一类特殊的可重定位文件，可以在程序加载或运行时读入内存并与程序动态链接。Linux 上的 `.so` 文件就是共享目标文件，也可以叫作共享库文件。
- 核心转储文件（Core Dump）。进程意外终止时，系统将进程的虚拟内存空间的内容保存下来形成的文件。

## 分析目标文件

> 真正了不起的程序员对自己程序的每一个字节都了如指掌。  ——佚名

在分析目标文件前，这里有一份有代表性的源代码，本文对目标文件的分析都基于这份代码。这份文件中设计的函数、变量能够较为全面的帮助理解目标文件的内容。

```c
int printf(const char *format, ...);

int global_init_var = 84;
int global_uninit_var;

void func1(int i) { printf("%d\n", i); }

int main(void) {
  static int static_var = 85;
  static int static_var2;
  int a = 1;
  int b;

  func1(static_var + static_var2 + a + b);

  return a;
}
```

### 创建目标文件

将上面的源代码保存为 `SimpleSection.c`，调用 `gcc -c SimpleSection.c -O1` 获得 `SimpleSection.o` 目标文件。

本文使用的 GCC 版本是 `gcc version 15.2.1 20250813 (GCC)`，Target `x86_64-pc-linux-gnu`。不同平台和编译器得到的目标文件内容可能会有所差异。

GCC 编译参数中的 `-c` 表示仅编译不链接，`-O1` 表示对代码进行一定程度的优化。

经过编译，从大小为 285 字节的 C 源代码文件得到了一个 1712 字节的目标文件。后面将分析每个字节在目标文件中的作用。

### 分析工具

目标文件是二进制文件，如果用常用的文本编辑器如 Vim、Emacs 打开，将会看到一长串不知所云的乱码。至于为什么是这样的乱码，这与字符编码规则有关，不在本文所讨论的范围之中。

分析二进制文件有专门的工具，如 `readelf` 和 `objdump`，一般包含在 `binutils` 软件包中。


- `readelf`。可以展示目标文件的文件头和结构，包含了 `size` 和 `nm` 的功能。
    - `-h`，文件头。
    - `-S`，段表。
    - `-t`，更为详细的段表信息。
    - `-s`，符号表。
    - `-j <section name>`，展示指定的段内容。
    - `-l`，运行时段分布。program headers。
    - `-e`，同 `-h -l -S`。
    - `-a`，同 `-h -l -S -s -r -d -V -A -I`。
    - `-x <段序号或段名>`，导出指定段的 16 进制数据。
- `objdump`，所有二进制工具所使用的底层工具。可以展示目标文件中的所有信息。它最有用的功能是反汇编代码段 `.text` 的机器指令。
    `-d`，反汇编代码段。



## 参考资料
- 程序员的自我修养——链接、装载与库，俞甲子、石凡、潘爱民