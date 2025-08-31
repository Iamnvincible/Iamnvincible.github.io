# 解读目标文件的每一个字节

## 目标文件

目标文件（Object File）是源代码经过编译但尚未链接的中间文件，是包含机器代码的二进制文件。在 Unix 平台上的目标文件遵循 ELF（Executable Linkable Format）文件格式存储。下面四种类型的文件使用 ELF 格式存储：

- 可重定位文件（Relocatable File）。Linux 上的 `.o` 文件，包含代码和数据，可以与其他可重定位文件链接成可执行文件或共享目标文件。静态链接库文件是可重定位文件的归档文件。
- 可执行文件（Executable File）。可以直接在目标平台上运行的文件，也可以称为“程序”，在 Linux 上一般没有扩展名，大家熟知的 `a.out` 就是可执行文件。
- 共享目标文件（Shared Object File）。一类特殊的可重定位文件，可以在程序加载或运行时读入内存并与程序动态链接。Linux 上的 `.so` 文件就是共享目标文件，也可以叫作共享库文件。
- 核心转储文件（Core Dump）。进程意外终止时，系统将进程的虚拟内存空间的内容保存下来形成的文件。

## 分析准备

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

将上面的源代码保存为 `SimpleSection.c`，编译获得 `SimpleSection.o` 目标文件。

```c
gcc -c SimpleSection.c -O1
```

本文使用的 GCC 版本是 `gcc version 15.2.1 20250813 (GCC)`，Target `x86_64-pc-linux-gnu`。不同平台和编译器得到的目标文件内容可能会有所差异。

GCC 编译参数中的 `-c` 表示仅编译不链接，`-O1` 表示对代码进行一定程度的优化。

经过编译，大小为 285 字节的 C 源代码文件产生了一个 **1712** 字节的目标文件，后面将介绍每个字节在目标文件中的作用。

### 分析工具

目标文件是二进制文件，如果用常用的文本编辑器如 Vim、Emacs 打开，将会看到一长串不知所云的乱码。至于为什么是这样的乱码，这与字符编码规则有关，不在本文所讨论的范围之中。

分析二进制文件有专门的工具，如 `readelf` 和 `objdump`，一般包含在 `binutils` 软件包中。

### ELF 文件格式

典型的 ELF 可重定位文件包括三个部分：文件头、段（Sections）、段表。这里先大致介绍三个部分的内容，后面会对这三个部分的数据展开分析。

#### 文件头
文件头描述了 ELF 的基本属性，在目标文件的最前面。在可执行程序运行前，装载程序会检查要运行的程序是否能够运行，首先就需要检查文件头，判断文件头是合法的 ELF 文件后才会进入后续步骤。在 Linux 中，ELF 文件头相关的定义在 `/usr/include/elf.h` 中。

这里以 64 位 ELF 文件头结构为例。`Elf64_Ehdr` 结构由 8 个 16 位、2 个 32 位、3 个 64位整数和 1 个 16 字节数组成员构成，整个文件头占用 64 字节。

```c
#define EI_NIDENT (16)

typedef uint16_t Elf64_Half;
typedef uint32_t Elf64_Word;
typedef uint64_t Elf64_Addr;
typedef uint64_t Elf64_Off;

typedef struct
{
  unsigned char e_ident[EI_NIDENT]; /* Magic number and other info */
  Elf64_Half    e_type;         /* Object file type */
  Elf64_Half    e_machine;      /* Architecture */
  Elf64_Word    e_version;      /* Object file version */
  Elf64_Addr    e_entry;        /* Entry point virtual address */
  Elf64_Off     e_phoff;        /* Program header table file offset */
  Elf64_Off     e_shoff;        /* Section header table file offset */
  Elf64_Word    e_flags;        /* Processor-specific flags */
  Elf64_Half    e_ehsize;       /* ELF header size in bytes */
  Elf64_Half    e_phentsize;        /* Program header table entry size */
  Elf64_Half    e_phnum;        /* Program header table entry count */
  Elf64_Half    e_shentsize;        /* Section header table entry size */
  Elf64_Half    e_shnum;        /* Section header table entry count */
  Elf64_Half    e_shstrndx;     /* Section header string table index */
 } Elf64_Ehdr;
 ```

#### 段

在文件头之后，是各个目标文件的段。段构成了目标文件的主要部分，包含了源代码文件编译后的代码、数据等内容。每个部分占用一个段，例如编译后的代码会放在 `.text` 代码段中，**已初始化的全局变量和静态变量**放在 `.data` 数据段，**未初始化或初始化为 0 的全局变量和静态变量**放在 `.bss` 段中。这些段的信息，包括段名称、占用空间等都由其段表项描述。


#### 段表

在 ELF 文件的最后，是描述段组织方式的段表（Section Header Table）。段表的结构也定义在 `/usr/include/elf.h` 中。这里以 64 位 ELF 文件的段表结构为例。段表就是若干个段表结构体的组合，每一个段用一个段表结构来描述，一项占用 64 字节。

```c
typedef uint64_t Elf64_Xword;
typedef struct
{
  Elf64_Word    sh_name;        /* Section name (string tbl index) */
  Elf64_Word    sh_type;        /* Section type */
  Elf64_Xword   sh_flags;       /* Section flags */
  Elf64_Addr    sh_addr;        /* Section virtual addr at execution */
  Elf64_Off     sh_offset;      /* Section file offset */
  Elf64_Xword   sh_size;        /* Section size in bytes */
  Elf64_Word    sh_link;        /* Link to another section */
  Elf64_Word    sh_info;        /* Additional section information */
  Elf64_Xword   sh_addralign;       /* Section alignment */
  Elf64_Xword   sh_entsize;     /* Entry size if section holds table */
} Elf64_Shdr;
```
## ELF 文件头

64 位的 ELF 文件头占用 64 字节，如果要查看原始数据内容，可以使用 `xxd` 命令查看文件前 64 字节内容。数出第一列是位置偏移（16 进制），一行展示 16 个字节。中间是具体数据（16 进制），右边是数据中的每一个字节在 ASCII 字符集中对应的符号，如果有对应会有可读字符，如果没有,就用 `.` 表示。

```
$ xxd -l 64 SimpleSection.o     
00000000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
00000010: 0100 3e00 0100 0000 0000 0000 0000 0000  ..>.............
00000020: 0000 0000 0000 0000 3003 0000 0000 0000  ........0.......
00000030: 0000 0000 4000 0000 0000 4000 0e00 0d00  ....@.....@.....
```

一般情况下，右侧对应的 ASCII 符号没有特别含义，只有少数情况下字符能够解析到有对应其含义的字符串。例如，ELF 文件头中的第 2 至 4 字节数据就对应 `ELF` 这个三个字符。为了能看到这些数据对应的含义，需要使用 `readelf` 来解析这些数据。

通过 `readelf -h SimpleSection.o` 读取文件头，得到下面的内容。

```
$ readelf -h SimpleSection.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          816 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         14
  Section header string table index: 13
```
### 文件头魔数

文件最前面的 16 个字节对应前面的文件头结构中的 16 字节字符数组。前 4 个字符是 `7f` `45` `4c` `46`。`7f` 是控制字符，后面三个刚好对应 `ELF`。这四个字节被称为 ELF 魔数。当一个可执行文件被执行时，操作系统首先会检查文件的魔数，数值正确才会加载，这可以避免不是可执行文件的文件被当作可执行文件错误执行。对操作系统来说，可执行文件与其他类型文件一样都是数据。数据在可执行文件中是机器指令，数据在常规文件中就只是数据，只有在数据能够对应合法指令时才能执行。如果任意的数据如果都当作执行执行，不知道会引起怎样的混乱。就像人类文字在合理的语法下才是有意义的文字，胡乱组合文字就只是呓语。

在魔数之后的数据代表 ELF 文件类型。第五字节 `02` 代表 64 位 ELF 文件，第六字节 `01` 代表字节序（小端），第七、八字节 `01 00` 是 ELF 的版本后，这些数值对应的信息在 `readelf` 的输出中有标注（Class，Data，OS/ABI，ABI Version）。第九字节之后的数据没有具体定义，可以当作扩展使用，一般是 0。

### 其他文件头信息

在魔数之后的数据（17 字节起），可对照文件头定义了解其含义。例如 `e_type` 值为 `0001`，表示可重定位文件。注意，数据是以小端字节序存放，分析时要转换字节序。 

其中比较重要的信息有：
- `Start of section headers`，段表在文件中的偏移位置。本例中，段表在文件第 816 字节开始。
- `Size of this header`，文件头长度，64 字节。
- `Size of section headers`，一个段表项的大小，64 字节。
- `Number of section headers`，段表中段表项目的数量，14 个。
- `Section header string table index`，表示段表名称的字符串的段在段表中的序号，定语较多，在后面介绍段之后再来理解。




---
### man
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