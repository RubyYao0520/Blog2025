做了一个小的实践，用C语言实现了128比特版本的AES算法。文章里面有很多公式，我实在调不好LaTex公式的渲染了，如果阅读起来不方便请见谅。



# 1.问题背景及相关知识

(1)算法提出背景

1976年，“公钥密码”的概念被提出。在此类密码中，公钥用于加密，私钥用于解密，加密解密使用两种不同的密钥。随后1978年发布的基于数论方法构造的RSA算法成为之后几十年密码学的研究方向。

针对RSA和DES算法的研究和计算机能力的不断提高，公钥密码体制的安全性逐渐受到威胁。数据加密标准DES不能满足官方和商业应用的安全要求，因此美国国家标准技术研究所NIST征集取代DES的新一代加密标准。经过五年的征集筛选，NIST最终将Rijndael算法确立为新的有效标准，以应对各种针对于RSA，DES算法的先进的密码分析技术，即高级加密标准AES。

(2)相关数学知识

a.有限域

有限域又称为伽瓦罗域，是具有有限个元素的域，集合中元素的个数称为域的阶。M阶域存在当且仅当m是某素数的幂，即存在某个整数$n$和素数$p$，使得$$\begin{align*}
\text{m = p}^{\text{n}}
\end{align*}$$。$p$称为有限域的特征。

b.扩展域

如果有限域的阶不是素数，则这样的有限域内的加法和乘法运算就不能用模整数加法和整数乘法模$p$表示。$m>1$的域称为扩展域。

在扩展域$$\begin{align*}
\text{GF}(2^{\text{m}})\end{align*}$$中元素并不是用整数表示的，而是用系数为$GF(2)$中元素的多项式表示。这个多项式最大的度为$m-1$

扩展域内的运算

假设$$A(x),B(x) \in \text{GF}(2^{\text{m}})$$

扩展域内的加法

$$ \begin{align*}
C(x) = A(x) + B(x) = \sum_{i=0}^{m-1} c_i x^i \quad \quad c_i \equiv (a_i + b_i) \text{mod} 2
\end{align*}$$

扩展域内的减法

$$\begin{align*}
C(x) = A(x) - B(x) = \sum_{i=0}^{m-1} c_i x^i \quad c_i \equiv (a_i - b_i) \text{mod} 2 \equiv (a_i + b_i) \text{mod} 2
\end{align*}$$

扩展域内的乘法

扩展域乘法主要在列混合步骤中使用，我们要将扩展域中的两个元素用多项式形式展开，然后使用多项式乘法规则将两个多项式相乘。

$$\begin{align*}
C(x) = A(x) \cdot B(x) \text{m} o dP(x)
\end{align*}$$

通常在多项式乘法中C(x)的度会大于m-1，因此需要对其进行化简。扩展域中进行化简的操作为：将这两个多项式相乘的结果除以一个不可约多项式，最后的结果就是最后的余数。

AES算法中使用的不可约多项式为:

$$\begin{align*}
P(x) = x^8 + x^4 + x^3 + x + 1
\end{align*}$$

# 2.算法原理

AES算法作为一种对称加密算法，加密/解密时通信双方使用单一密钥。而对称密码算法根据对明文消息加密方式的不同可分为两大类，分组密码和流密码。AES算法属于分组密码算法，它的输入分组、输出分组以及加密/解密过程中的中间分组都是128比特。密钥的长度K可以为128，192或256比特，本次实践中实现的是128比特的版本。AES加密算法的数据处理单元是字节，128比特的分组信息被分成16个字节按顺序被复制到一个的矩阵中，这个矩阵称为状态，AES所有变换都是基于状态的变换。AES的所有变换操作，包括字节替换、行变换、列混合和轮密钥添加，均围绕这个状态矩阵展开。这种矩阵化的表示方式不仅便于数学描述，也有利于硬件实现时的并行处理。

轮函数是AES加密的核心组件，通过多轮迭代实现数据的混淆（Confusion）与扩散（Diffusion）。在轮函数的每一轮迭代中包含四步变换，分别是：字节替换，行移位，列混合（最后一轮没有列混合）以及轮密钥添加，其作用即使通过简单的非线性变换、混合函数变换，将字节代换运算产生的非线性扩散达到充分混合，在每轮迭代中引入不同的密钥，这样就能以最简单的运算得到最好的加密效果，实现加密的有效性。

AES算法流程图如下：

![image-20250921155111272](http://image.slugyao.top/2025blog/aes/image-20250921155111272.png)

(1)字节替换

a. S盒的构造

首先对于任意输入的字节求逆元，如果x=0x00，则其逆元定义为0x00，否则在$$\begin{align*}
\text{GF}(2^8)
\end{align*}$$上计算x的逆元。

在域上$$\begin{align*}a \otimes b = 1
\end{align*}$$($$\otimes$$表示扩展域内的乘法)，则称b为a的逆元

对x的逆元应用在$$GF(2)$$上的仿射变换$$\begin{align*}Y = AX^{-1} + b
\end{align*}$$

得到Y，其中A是一个变换矩阵，b是一个常数

A是由循环矩阵构成的，具体内容为

<img src="http://image.slugyao.top/2025blog/aes/QianJianTec1758542989645.jpg" alt="QianJianTec1758542989645" width=200 />



经过上述变换之后得到的结果$Y$就是S盒中对应于$X$的输出

b.用S盒进行字节替换操作

明文中的每一个字节都会被另一个字节替换，替换规则来源于一个预先计算好的查找表，称为S盒。S盒是一个拥有256个字节元素的数组，可以定义为一个16*16的二维数组。

具体替换规则如下：

对于每个元素$A$，从存储角度都看作是一个八位的二进制数。算出前四位所代表的十六进制数$x$和后四位所代表的十六进制数$y$，在S盒中找到对应的$x$行$y$列的元素替换原来的字节

(2)行变换

行变换在状态矩阵State的每一行之间进行，具体操作为：对每一行及逆行循环移位，移动位数以字节为单位，第一行不变，第二行循环左移一个字节，第三行循环左移两个字节，第四行循环左移三个字节。

行变换是一种线性变换，目的是使密码信息充分混乱，提高非线性度。

(3)列混合

列混合是对状态矩阵State的每一列进行的一种线性变换，State的每一列有4个字节，即一个字$$\begin{align*}
    \{ b_0, b_1, b_2, b_3 \}
\end{align*}$$。将每一列视为$GF(2^{8})$上的一个多项式$$\begin{align*}
b(x) = b_3 x^3 + b_2 x^2 + b_1 x + b_0
\end{align*}$$，然后与一个固定的多项式$a(x)$进行模乘法，然后将所得的结果进行取模运算，模值为$$x^{4}+1$$。在$$\begin{align*}
\text{GF}(2^8)
\end{align*}$$上$$x^{4}+1$$不是不可约多项式，因此，用$b(x)$乘以任意一个四项式的运算不一定是可逆的，于是在AES中将$a(x)$设置成一个固定的值，以确保可以进行求逆运算。$$a(x)$$的表达式为:$$\begin{align*}a(x) = \{03\} x^3 + \{01\} x^2 + \{01\} x + \{02\}
\end{align*}$$。

(4)轮密钥加

将状态矩阵State中的每个字节与对应轮密钥的字节进行简单的按位异或操作，得到新一轮的密钥。

这样可以通过密钥扩展算法派生出来一系列子密钥，每一轮使用的子密钥都不相同。

(5)解密

对AES算法加密的密文进行解密，只需进行加密操作的逆过程即可。下面时比较重要的环节的算法原理。

a. 逆S盒的构造

对于每个字节$x$，先对$x$的进行逆仿射变换。仿射变换是$$\begin{align*}Y = AX^{-1} + b
\end{align*}$$，而逆仿射变换是$$\begin{align*}
E = A^{-1} (X + b)
\end{align*}$$得到结果$E$，其中$A^{-1}$是仿射变换中矩阵$A$的逆矩阵，$b$依然是常数63。然后对逆仿射变换的结果$E$求逆元，求逆元得到的结果就是字节$x$对应的逆S盒中的元素。

 

# 3.设计和实现过程

(1)字节替换

在字节替换前直接定义好S盒和逆S盒，方便后续直接使用

直接将S盒、逆S盒定义为16*16的char型数组

(2)行变换

行变换只需要简单的移位操作即可。

(3)列混合

在调用列混合函数之前，先定义了扩展域上的乘法函数

X2time为有限域上的乘2乘法的实现，X3time为有限域上乘3乘法的实现,以此类推定义好有限域上的乘法运算函数，方便在列混合时直接调用。

(4)轮密钥加

将状态矩阵State中的每个字节与对应轮密钥的字节进行简单的按位异或操作。

(5)解密过程

解密过程就是加密的逆过程。定义逆行移位、逆字节替换、逆列混合等函数。逆字节替换和加密时的字节替换类似，我们使用预先算好的逆S盒进行逆字节替换的操作。而逆行移位、逆列混合则是和加密时的操作完全相反即可。

(6)主函数设计

算法有两种执行模式，分别是使用预定义的NIST测试向量进行测试，以及用户自定义输入。

在用户自定义输入的情况下，用户可以选择加密或者解密模式，两种模式均需要用户输入密钥以及明文/密文进行加密/解密。

用户输入的密钥和明文/密文格式都是十六进制，每两位之间需要用空格间隔开，输入格式不正确时程序不会执行，并且输出错误提示信息。



完整代码如下

```c
//头文件
#pragma once

#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#ifdef __cplusplus
extern "C"{
#endif

    void encryptAES(unsigned char* out, const unsigned char* in, const unsigned char* key, const int keyLen);
    void decryptAES(unsigned char* out, const unsigned char* in, const unsigned char* key, const int keyLen);

#ifdef __cplusplus
}
#endif
```



```c
#include "aes.h"

// 在Windows系统中，为了正确显示中文，需要添加这个头文件
#ifdef _WIN32
#include <windows.h>
#endif

// 添加延迟函数所需的头文件
#ifdef _WIN32
#include <windows.h>
#else
#include <unistd.h>
#endif

// 延迟函数实现
void delay(int seconds) {
#ifdef _WIN32
    Sleep(seconds * 1000);
#else
    sleep(seconds);
#endif
}

const unsigned char sBox[16][16] = {
    0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5, 0x30, 0x01, 0x67, 0x2b, 0xfe, 0xd7, 0xab, 0x76,
    0xca, 0x82, 0xc9, 0x7d, 0xfa, 0x59, 0x47, 0xf0, 0xad, 0xd4, 0xa2, 0xaf, 0x9c, 0xa4, 0x72, 0xc0,
    0xb7, 0xfd, 0x93, 0x26, 0x36, 0x3f, 0xf7, 0xcc, 0x34, 0xa5, 0xe5, 0xf1, 0x71, 0xd8, 0x31, 0x15,
    0x04, 0xc7, 0x23, 0xc3, 0x18, 0x96, 0x05, 0x9a, 0x07, 0x12, 0x80, 0xe2, 0xeb, 0x27, 0xb2, 0x75,
    0x09, 0x83, 0x2c, 0x1a, 0x1b, 0x6e, 0x5a, 0xa0, 0x52, 0x3b, 0xd6, 0xb3, 0x29, 0xe3, 0x2f, 0x84,
    0x53, 0xd1, 0x00, 0xed, 0x20, 0xfc, 0xb1, 0x5b, 0x6a, 0xcb, 0xbe, 0x39, 0x4a, 0x4c, 0x58, 0xcf,
    0xd0, 0xef, 0xaa, 0xfb, 0x43, 0x4d, 0x33, 0x85, 0x45, 0xf9, 0x02, 0x7f, 0x50, 0x3c, 0x9f, 0xa8,
    0x51, 0xa3, 0x40, 0x8f, 0x92, 0x9d, 0x38, 0xf5, 0xbc, 0xb6, 0xda, 0x21, 0x10, 0xff, 0xf3, 0xd2,
    0xcd, 0x0c, 0x13, 0xec, 0x5f, 0x97, 0x44, 0x17, 0xc4, 0xa7, 0x7e, 0x3d, 0x64, 0x5d, 0x19, 0x73,
    0x60, 0x81, 0x4f, 0xdc, 0x22, 0x2a, 0x90, 0x88, 0x46, 0xee, 0xb8, 0x14, 0xde, 0x5e, 0x0b, 0xdb,
    0xe0, 0x32, 0x3a, 0x0a, 0x49, 0x06, 0x24, 0x5c, 0xc2, 0xd3, 0xac, 0x62, 0x91, 0x95, 0xe4, 0x79,
    0xe7, 0xc8, 0x37, 0x6d, 0x8d, 0xd5, 0x4e, 0xa9, 0x6c, 0x56, 0xf4, 0xea, 0x65, 0x7a, 0xae, 0x08,
    0xba, 0x78, 0x25, 0x2e, 0x1c, 0xa6, 0xb4, 0xc6, 0xe8, 0xdd, 0x74, 0x1f, 0x4b, 0xbd, 0x8b, 0x8a,
    0x70, 0x3e, 0xb5, 0x66, 0x48, 0x03, 0xf6, 0x0e, 0x61, 0x35, 0x57, 0xb9, 0x86, 0xc1, 0x1d, 0x9e,
    0xe1, 0xf8, 0x98, 0x11, 0x69, 0xd9, 0x8e, 0x94, 0x9b, 0x1e, 0x87, 0xe9, 0xce, 0x55, 0x28, 0xdf,
    0x8c, 0xa1, 0x89, 0x0d, 0xbf, 0xe6, 0x42, 0x68, 0x41, 0x99, 0x2d, 0x0f, 0xb0, 0x54, 0xbb, 0x16
};

const unsigned char inv_sBox[16][16] = {
    0x52, 0x09, 0x6a, 0xd5, 0x30, 0x36, 0xa5, 0x38, 0xbf, 0x40, 0xa3, 0x9e, 0x81, 0xf3, 0xd7, 0xfb,
    0x7c, 0xe3, 0x39, 0x82, 0x9b, 0x2f, 0xff, 0x87, 0x34, 0x8e, 0x43, 0x44, 0xc4, 0xde, 0xe9, 0xcb,
    0x54, 0x7b, 0x94, 0x32, 0xa6, 0xc2, 0x23, 0x3d, 0xee, 0x4c, 0x95, 0x0b, 0x42, 0xfa, 0xc3, 0x4e,
    0x08, 0x2e, 0xa1, 0x66, 0x28, 0xd9, 0x24, 0xb2, 0x76, 0x5b, 0xa2, 0x49, 0x6d, 0x8b, 0xd1, 0x25,
    0x72, 0xf8, 0xf6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xd4, 0xa4, 0x5c, 0xcc, 0x5d, 0x65, 0xb6, 0x92,
    0x6c, 0x70, 0x48, 0x50, 0xfd, 0xed, 0xb9, 0xda, 0x5e, 0x15, 0x46, 0x57, 0xa7, 0x8d, 0x9d, 0x84,
    0x90, 0xd8, 0xab, 0x00, 0x8c, 0xbc, 0xd3, 0x0a, 0xf7, 0xe4, 0x58, 0x05, 0xb8, 0xb3, 0x45, 0x06,
    0xd0, 0x2c, 0x1e, 0x8f, 0xca, 0x3f, 0x0f, 0x02, 0xc1, 0xaf, 0xbd, 0x03, 0x01, 0x13, 0x8a, 0x6b,
    0x3a, 0x91, 0x11, 0x41, 0x4f, 0x67, 0xdc, 0xea, 0x97, 0xf2, 0xcf, 0xce, 0xf0, 0xb4, 0xe6, 0x73,
    0x96, 0xac, 0x74, 0x22, 0xe7, 0xad, 0x35, 0x85, 0xe2, 0xf9, 0x37, 0xe8, 0x1c, 0x75, 0xdf, 0x6e,
    0x47, 0xf1, 0x1a, 0x71, 0x1d, 0x29, 0xc5, 0x89, 0x6f, 0xb7, 0x62, 0x0e, 0xaa, 0x18, 0xbe, 0x1b,
    0xfc, 0x56, 0x3e, 0x4b, 0xc6, 0xd2, 0x79, 0x20, 0x9a, 0xdb, 0xc0, 0xfe, 0x78, 0xcd, 0x5a, 0xf4,
    0x1f, 0xdd, 0xa8, 0x33, 0x88, 0x07, 0xc7, 0x31, 0xb1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xec, 0x5f,
    0x60, 0x51, 0x7f, 0xa9, 0x19, 0xb5, 0x4a, 0x0d, 0x2d, 0xe5, 0x7a, 0x9f, 0x93, 0xc9, 0x9c, 0xef,
    0xa0, 0xe0, 0x3b, 0x4d, 0xae, 0x2a, 0xf5, 0xb0, 0xc8, 0xeb, 0xbb, 0x3c, 0x83, 0x53, 0x99, 0x61,
    0x17, 0x2b, 0x04, 0x7e, 0xba, 0x77, 0xd6, 0x26, 0xe1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0c, 0x7d
};

const unsigned char Rcon[16] = {
    0x00, 0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1b, 0x36, 0xd8, 0xab, 0x4d, 0x9a, 0x2f
}; // 轮常量表

void SubBytes(unsigned char *in, const unsigned char table[16][16]) {
    // 字节替换
    for (int i = 0; i < 16; i++)
        in[i] = table[in[i] >> 4][in[i] & 0x0F];
}

void ShiftRows(unsigned char *in) {
    // 第0行不变
    // 第1行左移1位
    unsigned char temp = in[1];
    in[1] = in[5];
    in[5] = in[9];
    in[9] = in[13];
    in[13] = temp;

    // 第2行左移2位
    temp = in[2];
    in[2] = in[10];
    in[10] = temp;
    temp = in[6];
    in[6] = in[14];
    in[14] = temp;

    // 第3行左移3位
    temp = in[3];
    in[3] = in[15];
    in[15] = in[11];
    in[11] = in[7];
    in[7] = temp;
}

void InvShiftRows(unsigned char *in) {
    // 第0行不变

    // 第1行右移1位 (相当于左移3位)
    unsigned char temp = in[13];
    in[13] = in[9];
    in[9] = in[5];
    in[5] = in[1];
    in[1] = temp;

    // 第2行右移2位 (相当于左移2位)
    temp = in[2];
    in[2] = in[10];
    in[10] = temp;
    temp = in[6];
    in[6] = in[14];
    in[14] = temp;

    // 第3行右移3位 (相当于左移1位)
    temp = in[7];
    in[7] = in[11];
    in[11] = in[15];
    in[15] = in[3];
    in[3] = temp;
}


unsigned char x2time(unsigned char a) {
    // 有限域*2乘法
    //如果a的最高位为1，则异或0x1b，否则异或0x00
    unsigned char res = (a << 1) ^ ((a & 0x80) ? 0x1b : 0x00);
    return res;
}

unsigned char x3time(unsigned char a) {
    // 3:0011
    return (x2time(a) ^ a);
}

unsigned char x4time(unsigned char a) {
    // 4:0100
    return (x2time(x2time(a)));
}

unsigned char x8time(unsigned char a) {
    // 8:1000
    return (x2time(x2time(x2time(a))));
}

unsigned char x9time(unsigned char a) {
    // 9:1001
    return (x8time(a) ^ a);
}

unsigned char xBtime(unsigned char a) {
    // B:1011
    return (x8time(a) ^ x3time(a));
}

unsigned char xDtime(unsigned char a) {
    // D:1101
    return (x8time(a) ^ x4time(a) ^ a);
}

unsigned char xEtime(unsigned char a) {
    // E:1110
    return (x8time(a) ^ x4time(a) ^ x2time(a));
}

void MixColumn(unsigned char *in) {
    // 列混合
    unsigned char tmp[4] = {0};
    for (int i = 0; i < 4; i++, in += 4) {
        tmp[0] = x2time(in[0]) ^ x3time(in[1]) ^ in[2] ^ in[3];
        tmp[1] = in[0] ^ x2time(in[1]) ^ x3time(in[2]) ^ in[3];
        tmp[2] = in[0] ^ in[1] ^ x2time(in[2]) ^ x3time(in[3]);
        tmp[3] = x3time(in[0]) ^ in[1] ^ in[2] ^ x2time(in[3]);
        memcpy(in, tmp, 4);
    }
}

void InvMixColumn(unsigned char *in) {
    // 逆列混合
    unsigned char tmp[4] = {0};
    for (int i = 0; i < 4; i++, in += 4) {
        tmp[0] = xEtime(in[0]) ^ xBtime(in[1]) ^ xDtime(in[2]) ^ x9time(in[3]);
        tmp[1] = x9time(in[0]) ^ xEtime(in[1]) ^ xBtime(in[2]) ^ xDtime(in[3]);
        tmp[2] = xDtime(in[0]) ^ x9time(in[1]) ^ xEtime(in[2]) ^ xBtime(in[3]);
        tmp[3] = xBtime(in[0]) ^ xDtime(in[1]) ^ x9time(in[2]) ^ xEtime(in[3]);
        memcpy(in, tmp, 4);
    }
}

void AddRoundKey(unsigned char *in, const unsigned char *key) {
    for (int i = 0; i < 16; i++)
        in[i] ^= key[i];
}

void rotWord(unsigned char *in) {
    // 将一个字进行左移循环一个单位
    unsigned char tmp = in[0];
    memcpy(in, in + 1, 3);
    in[3] = tmp;
}

void subWord(unsigned char *in) {
    // 每一个字进行字节替换
    for (int i = 0; i < 4; i++)
        in[i] = sBox[in[i] >> 4][in[i] & 0x0F];
}

// 存储扩展后的轮密钥数组
static unsigned char roundKey[240] = {0};

void SetKey(const unsigned char *key, const int Nk) {
    // Nr为加密轮数，计算公式为Nk + 6
    int Nr = Nk + 6;
    // 将初始密钥复制到轮密钥数组的起始位置
    // 每字4字节，所以复制4*Nk个字节
    memcpy(roundKey, key, 4 * Nk);

    // 生成剩余的轮密钥
    // 总共需要生成(Nr+1)*4个字的轮密钥
    for (int i = Nk; i < (Nr + 1) * 4; i++) {
        unsigned char buffer[4] = {0};

        // 复制前一个字到缓冲区
        memcpy(buffer, roundKey + (i - 1) * 4, 4);

        if (i % Nk == 0) {
            // 当i是Nk的倍数时
            // 循环左移一个字节
            rotWord(buffer);
            // 字节替换
            subWord(buffer);
            buffer[0] ^= Rcon[i / Nk];
        }

        // 与前Nk个字异或生成新字
        for (int j = 0; j < 4; j++) {
            roundKey[4 * i + j] = buffer[j] ^ roundKey[(i - Nk) * 4 + j];
        }
    }
}

void encryptAES(unsigned char *out, const unsigned char *in, const unsigned char *key, const int keyLen) {
    int Nk = keyLen >> 2;
    int Nr = Nk + 6;
    SetKey(key, Nk);
    memcpy(out, in, 16);
    AddRoundKey(out, roundKey);
    for (int i = 1; i <= Nr; i++) {
        // 进行Nr轮变换，最后一个Nr轮不进行列混合
        SubBytes(out, sBox);
        ShiftRows(out);
        if (i != Nr)
            MixColumn(out);
        AddRoundKey(out, roundKey + 16 * i);
    }
}

void decryptAES(unsigned char *out, const unsigned char *in, const unsigned char *key, const int keyLen) {
    int Nk = keyLen >> 2;
    int Nr = Nk + 6;
    SetKey(key, Nk);
    memcpy(out, in, 16);
    AddRoundKey(out, roundKey + Nr * 16);
    for (int i = Nr - 1; i >= 0; i--) {
        InvShiftRows(out);
        SubBytes(out, inv_sBox);
        AddRoundKey(out, roundKey + 16 * i);
        if (i != 0)
            InvMixColumn(out);
    }
}

int main() {
#ifdef _WIN32
    SetConsoleOutputCP(65001);
    SetConsoleCP(65001);
#endif

    // 程序控制变量
    char continueProgram = 'y'; // 控制是否继续执行程序
    unsigned char key[16] = {0}; // 存储16字节密钥
    unsigned char input[16] = {0}; // 存储输入数据(明文或密文)
    unsigned char output[16] = {0}; // 存储输出数据(密文或明文)
    int mode; // 操作模式(1-加密, 2-解密)
    int useTestVector; // 输入方式选择(1-NIST测试向量, 2-自定义输入)

    // 预定义NIST 128位AES测试向量 - 用于验证AES实现的正确性
    const unsigned char nistKey[16] = {
        0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f
    };
    // 明文
    const unsigned char nistPlaintext[16] = {
        0x00, 0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88, 0x99, 0xaa, 0xbb, 0xcc, 0xdd, 0xee, 0xff
    };
    // 密文
    const unsigned char nistCiphertext[16] = {
        0x69, 0xc4, 0xe0, 0xd8, 0x6a, 0x7b, 0x04, 0x30, 0xd8, 0xcd, 0xb7, 0x80, 0x70, 0xb4, 0xc5, 0x5a
    };

    // 显示程序标题
    printf("===== AES 128位加密工具 =====\n");

    // 主循环 - 支持多次加密/解密操作
    while (continueProgram == 'y' || continueProgram == 'Y') {
        // 选择使用测试向量还是自定义输入
        printf("\n请选择输入方式:\n");
        printf("1. 使用NIST测试向量\n");
        printf("2. 自定义输入\n");
        printf("请输入选择 (1/2): ");
        scanf("%d", &useTestVector);
        getchar();

        if (useTestVector == 1) {
            // 使用NIST测试向量模式
            memcpy(key, nistKey, 16);
            memcpy(input, nistPlaintext, 16);
            printf("已加载NIST测试向量:\n");
            printf("密钥: ");
            for (int i = 0; i < 16; i++) {
                printf("%02x ", key[i]);
            }
            printf("\n");
            // 执行加密操作
            encryptAES(output, input, key, 16);
            // 输出密文并与标准密文比较
            printf("加密结果 (十六进制): ");
            int match = 1;
            for (int i = 0; i < 16; i++) {
                printf("%02x ", output[i]);
                if (output[i] != nistCiphertext[i]) {
                    match = 0;
                }
            }
            printf("\n");
            if (match) {
                printf("加密结果与NIST标准密文匹配!\n");
            } else {
                printf("加密结果与NIST标准密文不匹配!\n");
                printf("标准密文: ");
                for (int i = 0; i < 16; i++) {
                    printf("%02x ", nistCiphertext[i]);
                }
                printf("\n");
            }
            // 执行解密测试
            unsigned char decrypted[16] = {0};
            decryptAES(decrypted, nistCiphertext, key, 16);
            // 输出解密结果并与原始明文比较
            printf("解密结果 (十六进制): ");
            match = 1;
            for (int i = 0; i < 16; i++) {
                printf("%02x ", decrypted[i]);
                if (decrypted[i] != nistPlaintext[i]) {
                    match = 0;
                }
            }
            printf("\n");
            // 显示解密结果比较信息
            if (match) {
                printf("解密结果与原始明文匹配!\n");
            } else {
                printf("解密结果与原始明文不匹配!\n");
            }
        } else if (useTestVector == 2) {
            // 自定义输入模式
            // 选择加密或解密模式
            printf("\n请选择操作模式:\n");
            printf("1. 加密\n");
            printf("2. 解密\n");
            printf("请输入选择 (1/2): ");
            scanf("%d", &mode);
            getchar();

            printf("请输入16字节密钥 (十六进制，每两位用空格分隔): ");
            char hexInput[64];
            fgets(hexInput, sizeof(hexInput), stdin);
            hexInput[strcspn(hexInput, "\n")] = '\0'; // 去除换行符

            // 解析密钥
            int byteCount = 0;
            char *token = strtok(hexInput, " ");
            while (token != NULL && byteCount < 16) {
                if (strlen(token) == 2) {
                    sscanf(token, "%2hhx", &key[byteCount]);
                    byteCount++;
                }
                token = strtok(NULL, " ");
            }
            // 判断密钥格式是否正确
            if (byteCount != 16) {
                printf("密钥格式错误，请确保输入了16个字节的十六进制值，每两位用空格分隔。\n");
                continue; // 重新开始循环
            }

            // 根据选择的模式处理输入
            if (mode == 1) {
                // 加密模式 - 输入并解析明文
                printf("请输入要加密的明文 (16字节，十六进制，每两位用空格分隔): ");
                fgets(hexInput, sizeof(hexInput), stdin);
                hexInput[strcspn(hexInput, "\n")] = '\0';

                // 解析明文
                byteCount = 0;
                token = strtok(hexInput, " ");
                while (token != NULL && byteCount < 16) {
                    if (strlen(token) == 2) {
                        sscanf(token, "%2hhx", &input[byteCount]);
                        byteCount++;
                    }
                    token = strtok(NULL, " ");
                }

                // 验明文格式是否正确
                if (byteCount != 16) {
                    printf("明文格式错误，请确保输入了16个字节的十六进制值，每两位用空格分隔。\n");
                    continue;
                }

                // 执行加密操作
                encryptAES(output, input, key, 16);

                // 输出密文结果
                printf("加密结果 (十六进制): ");
                for (int i = 0; i < 16; i++) {
                    printf("%02x ", output[i]);
                }
                printf("\n");
            } else if (mode == 2) {
                // 解密模式 - 输入并解析密文
                printf("请输入要解密的密文 (16字节，十六进制，每两位用空格分隔): ");
                fgets(hexInput, sizeof(hexInput), stdin);
                hexInput[strcspn(hexInput, "\n")] = '\0';
                // 解析密文
                byteCount = 0;
                token = strtok(hexInput, " ");
                while (token != NULL && byteCount < 16) {
                    if (strlen(token) == 2) {
                        sscanf(token, "%2hhx", &input[byteCount]);
                        byteCount++;
                    }
                    token = strtok(NULL, " ");
                }
                // 判断密文格式是否正确，如果错误输出提示信息
                if (byteCount != 16) {
                    printf("密文格式错误，请确保输入了16个字节的十六进制值，每两位用空格分隔。\n");
                    continue;
                }
                // 执行解密操作
                decryptAES(output, input, key, 16);
                printf("解密结果 (十六进制): ");
                for (int i = 0; i < 16; i++) {
                    printf("%02x ", output[i]);
                }
                printf("\n");

            } else {
                // 无效的模式选择
                printf("无效的选择，请重新输入。\n");
                continue;
            }
        } else {
            // 无效的输入方式选择
            printf("无效的选择，请重新输入。\n");
            continue;
        }

        // 询问是否继续执行程序
        printf("\n是否继续? (y/n): ");
        continueProgram = getchar();
        getchar(); // 吸收换行符
    }

    // 程序退出提示
    printf("\n程序已退出，感谢使用！\n");
    return 0;
}

```
