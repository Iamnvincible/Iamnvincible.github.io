---
layout: post
title: "MD5 算法的不完全实现"
categories: Algorithm
tags: md5
permalink: /4/
---

MD5 算法是一种已经被证明不安全的消息摘要算法，该算法在 [RFC 1321](https://datatracker.ietf.org/doc/html/rfc1321) 规范中被实现。


在 GNU/Linux 系统中，命令 `md5sum` 可以完成校验文件或输入字符串的工作，该命令包含在 [GNU core utilities](https://www.gnu.org/software/coreutils/) 命令集合中，MD5 算法的核心代码在 [gnulib](https://git.savannah.gnu.org/gitweb/?p=gnulib.git)。

实现 MD5 算法的代码并不复杂，动手实践一遍可以提高动手能力和对规范的理解能力。本文依照规范，尝试实现对简单字符串消息摘要计算。

## 概述

MD5 消息摘要算法接收一段任意长度的输入，输出一个 128 位的“指纹”或“消息摘要”。

一般认为，两段不同的消息不太可能会有一样的消息摘要。同时，也不容易在确定一个目标消息摘要后生成一段会得到这个消息摘要的消息。MD5 算法一般用于数字签名程序，这些程序需要在安全地“压缩”一个大文件后再执行公私钥体系的加密。

MD5 算法在 32 位计算机上运行极快，也不需要替换表，算法代码非常紧凑。

MD5 算法是 MD4 消息摘要算法的扩展。由于 MD5 算法采用了一个更加可靠的设计，所以运行速度会比 MD4 算法稍慢。

## 术语和符号
本文中，“字”是 32 位长度，“字节”是 8 位长度。接收到以“位”组成的消息序列时，可以按照大端字节序规则来将每 8 个位组合成一个字节，同样也可以按小端字节序规则将每 8 个位组成一个字节。

- 下标：`x_i`，表示 i 是 x 的下标。下标是表达式时用括号括起来，如 x_{i+1}。
- 上标：`x^i`，表示 i 是 x 的上标，表示 x 的 i 次幂。
- 循环左移：`X <<< s`，表示 X 向左移动 s 位，位移动是首位循环的。
- 按位取反：`not(X)`。
- 按位或：`X v Y`。
- 按位异或：`x xor Y`。

## MD5 算法描述

假设有一个 b 位长度的消息 m 作为输入，算法计算这段消息的摘要。
- b 可以是任意非负整数，可以是很大的数。
- b 可以是 0。
- b 不一定能被 8 整除。

那么消息可以写作：
```
m_0 m_1 ... m_{b-1}
```
### 第一步：填充消息到特定长度
将消息填充至其位长度能够在被 512 取模时余 448。即使消息长度刚好被 512 整除余 448 也需要做一次填充。

填充规则：
1. 向消息追加一位 `1`。
2. 继续追加若干位 `0`，直到消息长度能被 512 取模余 448。

按此规则，消息至少填充 1 位，最多填充 512 位。

### 第二步：填充消息长度

向上一步填充完成后的消息后填充一个 64 位整数——以 64 位表示的消息长度 b（以位记）。如果消息长度大于 64 位，只取消息长度的低 64 位。附加这个整数时，低有效位在前，即以小端字节序规则，按字节附加。



> 完成两步填充后，消息长度将刚好能被 512 整除。即能以每组 16 个字（ 16个 32 位整数）的方式，分成整数组。

前两步的 C 语言实现如下。注意：代码中假定处理的消息为字符串，所以处理消息长度时按字节长度处理，但注释中以位说明。

```c
/**
 * @brief  计算特定长度的消息需要填充多少字节
 * @note   
 * @param  message_length: 消息长度
 * @retval 需要填充的字节数
 */
int caculate_padding_length(size_t message_length)
{
    // 512 bits = 64 bytes
    int mod = message_length % 64; //长度对 512 取模
    int padding_bytes = 0;         // 要填充的字节数
    // 448 bits = 56 bytes
    // 余数与 448 比较
    if (mod >= 56)
    {
        // 余数大于 448 时，先补长度到刚好能被 512 整除的长度，再补 448 位
        padding_bytes = 64 - mod + 56;
    }
    else
    {
        padding_bytes = 56 - mod;
    }
    //填充 64 位长度的整数
    padding_bytes += 8;
    return padding_bytes;
}

/**
 * @brief  为特定长度消息创建填充的数据
 * @note   
 * @param  message_length: 消息长度
 * @retval 填充消息数据，调用方负责内存释放
 */
unsigned char *append_bytes(size_t message_length)
{
    int padding_length = caculate_padding_length(message_length);
    unsigned char *text = NULL; //填充在消息后的字符串
    text = (unsigned char *)malloc(sizeof(unsigned char) * padding_length);
    memset(text, 0, padding_length);
    int i;
    //第一步填充
    for (i = 0; i < padding_length - 8; i++)
    {
        if (i == 0)
        {
            // 填充的第一个字节为 1000 0000，第一位是1，后面附加 0
            text[i] = 0x80;
        }
        else
        {
            text[i] = 0x00;
        }
    }
    //第二步填充
    size_t message_length_bits = message_length * 8;
    memcpy(text + i, (unsigned char *)(&message_length_bits), 8);
    return text;
}
```

