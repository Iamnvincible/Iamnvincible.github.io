---
layout: post
title: "字节序"
categories: Computer architecture
tags: Endianness Linux
permalink: /2/
---

## 什么是“字”？
字是表示计算机数据单位的术语，是计算机一次处理事务的一个固定长度的位组。当要处理的数据是 32 位整数时，一个字就是 32 位，4 个字节。例如，处理 32 位整数的加法时，计算机一次需要输入 4 个字节的数据作为参与运算的一个“字”。

> 这里对“字”的解释不够准确，但有助于理解字节序。

## 字节序

字节序表示一个由多个字节组成的字中字节的排列顺序。字节序有两种，一种是按照字面值排列，另一种与之相反。给出一个字所在的连续内存，需要按照预定的顺序读取字中的字节来组成字。通常，以两种顺序分别读取字节成字，会得到完全不同的值。

例如：给出一个由 4 个字节组成的字（一个 32 位整数）在内存中的十六进制表示，`12345678`。其中，前面（左边）的字节所在的内存地址为低地址，后面（右边）的字节所在的内存地址为高地址。按照人类读写习惯，这个字应当读作 `0x12345678`。但如果按照以人类读写习惯相反的顺序读取，这块内存所表示的字就是 `0x78563412`。注意，读取时以单个字节（8 位）为单位，所以结果不是 `87654321`。

## 大端序

大端字节序是一个与人类读写习惯一致的字节序。字中连续字节先读到的字节（内存低地址）作为数值的高位，后读到的（内存高地址）作为数值的低位。即数值的高位放在内存低地址处，低位放在高地址处。

例如：一个 4 个字节的字的内存首地址为 `0x100`，占用了 `0x100~0x103` 的地址空间。这个字的值为十六进制数字 `0x12345678`，高位是 `0x12`,低位是 `0x78`。按照大端字节序的规则，这 4 个字节地址中分别存放的数据分别是：
- `0x100`:`0x12`
- `0x101`:`0x34`
- `0x102`:`0x56`
- `0x103`:`0x78`
  
## 小端序
小端字节序将先从字中读出的字节（内存低地址）作为数值的低位，后读到的字节（内存高地址）作为数值的高位。按照小端字节序的规则，数字 `0x12345678` 在内存中应当表示为：
- `0x100`:`0x78`
- `0x101`:`0x56`
- `0x102`:`0x34`
- `0x103`:`0x12`


## 在树莓派上的示例

```c
int main(){
    unsigned int value = 0x12345678;
    printf("Hex value = %x\n",value);
    unsigned int* pv = &value;
    unsigned char* pc = (unsigned char*)pv;
    for(int i=0;i<sizeof(value);i++){
      printf("Hex vaule = %x , address = %p\n",pc[i],pc+i);
    }
    return 0;
}
```
输出如下：

```bash
Hex value = 12345678
Hex vaule = 78 , address = 0xffffff3caad0 #内存的低地址区域存放数值的低位
Hex vaule = 56 , address = 0xffffff3caad1
Hex vaule = 34 , address = 0xffffff3caad2
Hex vaule = 12 , address = 0xffffff3caad3

$ cat /proc/device-tree/model
Raspberry Pi 3 Model B Rev 1.2% 

$ lscpu
Architecture:            aarch64
  CPU op-mode(s):        32-bit, 64-bit
  Byte Order:            Little Endian # 小端字节序CPU
CPU(s):                  4
  On-line CPU(s) list:   0-3

```
## 什么时候会用到字节序
存储一个整数时，计算机会找一块合适大小的内存空间来保存。当以一个整数的方式来读取一块内存时，这块是一个整体，不关心字节的存储顺序。而以单个字节的方式来访问这个整数所在的地址空间时，将从低地址开始，依次读取每个字节中所存储的数据。对于小端字节序的计算机，先读到的字节（低地址）是这个整数低位的数据。

可以发现，在同一台计算机上处理数据时，我们不需要考虑字节序，因为写入和读取用的是同一种规则。但是，当需要和其他计算机通信或编写跨平台程序时，就需要考虑字节序。常用的个人计算机所采用的 CPU 一般是小端字节序，而 PowerPC 处理器采用的是大端字节序。为了确保不同字节序处理器能够通过网络正确的处理信息，TCP/IP 协议定义大端序为网络字节序。

## 字节序转换
在网络编程时，经常需要将网络中获得的数据转换为主机字节序。在 Linux 中通常使用 `arpa/inet.h` 头文件下的 4 个转换函数完成字节序的转换，在不同字节序处理器上可以通用。

```c
#include <arpa/inet.h>
// 主机字节序转换为网络字节序（无符号32位、16位整数）
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
// 网络字节序转换为主机字节序（无符号32位、16位整数）
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

### 64 位？
有时候，可能需要将 64 位整数转换字节序，下面方法来自[stack overflow](https://stackoverflow.com/questions/3022552/is-there-any-standard-htonl-like-function-for-64-bits-integers-in-c)。
```c
#define htonll(x) ((1==htonl(1)) ? (x) : ((uint64_t)htonl((x) & 0xFFFFFFFF) << 32) | htonl((x) >> 32))
#define ntohll(x) ((1==ntohl(1)) ? (x) : ((uint64_t)ntohl((x) & 0xFFFFFFFF) << 32) | ntohl((x) >> 32))
```
这两个宏非常巧妙，首先判断当前计算机字节序，需要转换时先后转换前后 32 位数字，再将先后 32 位数字通过“或”运算组成 64 位结果。