---
layout: post
title: 详解大端模式和小端模式
summary: 很久之前在学习网络编程时就接触到大端模式和小端模式，也记忆过很多次，但是每次过一段时间就忘了，又得重新看。为了方便以后资料的查找，同时也加深记忆，将大端模式和小端模式通过文章写出来。
categories: [network]
tags: [network]
published: true
---

# {{ page.title }} #
{{ page.summary }}

### 一、大端模式和小端模式的来源 ###
关于大端小端名词的由来，有一个有趣的故事，来自于Jonathan Swift的《格利佛游记》：Lilliput和Blefuscu这两个强国在过去的36个月中一直在苦战。战争的原因：大家都知道，吃鸡蛋的时候，原始的方法是打破鸡蛋较大的一端，可是那时的皇帝的祖父由于小时侯吃鸡蛋，按这种方法把手指弄破了，因此他的父亲，就下令，命令所有的子民吃鸡蛋的时候，必须先打破鸡蛋较小的一端，违令者重罚。然后老百姓对此法令极为反感，期间发生了多次叛乱，其中一个皇帝因此送命，另一个丢了王位，产生叛乱的原因就是另一个国家Blefuscu的国王大臣煽动起来的，叛乱平息后，就逃到这个帝国避难。据估计，先后几次有11000余人情愿死也不肯去打破鸡蛋较小的端吃鸡蛋。这个其实讽刺当时英国和法国之间持续的冲突。Danny Cohen一位网络协议的开创者，第一次使用这两个术语指代字节顺序，后来就被大家广泛接受。

### 二、大端模式 ###
Big-Endian(大端模式)：就是将数字高位字节放在内存的低地址，数字低字节放在内存的高地址，比如数字0x12 34 56 78(12是数字的高位字节，78是数字的低位字节)使用大端模式在内存中的表示为：  
低地址 -----------》 高地址  
0x12 | 0x34 | 0x56 | 0x78  

### 三、小端模式 ###
Little-Endian(小端模式)：就是将数字高位字节放在内存的高地址，数字低字节放在内存的低地址，比如数字0x12 34 56 78(12是数字的高位字节，78是数字的低位字节)使用小端模式在内存中的表示为：  
低地址 ------------》 高地址  
0x78 | 0x56 | 0x34 | 0x12  

### 四、为什么有大端和小端之分 ###
这是因为在计算机系统中，我们是以字节为单位的，每个地址单元都对应着一个字节，一个字节为8bit。但是在C语言中除了8bit的char之外，还有16bit的short型，32bit的long型（要看具体的编译器），另外，对于位数大于8位的处理器，例如16位或者32位的处理器，由于寄存器宽度大于一个字节，那么必然存在着一个如果将多个字节安排的问题。因此就导致了大端存储模式和小端存储模式。例如一个16bit的short型x，在内存中的地址为0x0010，x的值为0x1122，那么0x11为高字节，0x22为低字节。对于大端模式，就将0x11放在低地址中，即0x0010中，0x22放在高地址中，即0x0011中。小端模式，刚好相反。我们常用的X86结构是小端模式，而KEIL C51则为大端模式。很多的ARM，DSP都为小端模式。有些ARM处理器还可以由硬件来选择是大端模式还是小端模式。

### 五、常见CPU的字节序 ###
大端模式构架CPU： PowerPC、IBM、Sun  
小端模式构架CPU： X86、DEC  
注意：ARM即可以工作在大端模式，也可以工作在小端模式  

### 六、常见文件的字节序 ###
Adobe PS:   大端模似  
BMP:        小端模式  
GIF:        小端模式  
JPEG:       大端模式  
RTF:        小端模式  

### 七、判断机器字节序 ###
既然已经了接了大端模式和小端模式只是两种不同存储字节的方式，那么如果通过程序判断机器的字节序呢，下面程序中给出两种方式判断电脑的字节序：  

{%highlight c%}
#include <stdio.h>
/*
 *  方法1：通过将int强制转换成char单字节，因为强制转换成char能得到int的
 *          低地址部分，通过判断这个低地址部分来判断机器字节序
 *
 */
bool is_big_endian_1(){
    int a = 0x1234;
    char b;
    b = *(char*)&a;

    if (0x12 == b)
        return true;

    return false;
}

/*
 *  方法2：利用联合体union的存放顺序是所有成员都从低地址开始存放，然后获取
 *          低地址部分判断机器字节序(关于union的妙用将会在后面的文字中介绍)
 *
 */
 bool is_big_endian_2(){
     union NUM{
        int a;
        char b;
     }num;

     num.a = 0x1234;

     if (0x12 == num.b)
         return true;

     return false;
 }

int main(int argc, char **argv){
    
    printf("start test is_big_endian_1:");
    if ( is_big_endian_1() )
        printf("your computer is Big-Endian...\n");
    else
        printf("your computer is Little-Endian...\n");

    printf("start test is_big_endian_2:");
    if ( is_big_endian_2() )
        printf("your computer is Big-Endian...\n");
    else
        printf("your computer is Little-Endian...\n");
    return 0;
}
{%endhighlight%}

### 八、主机字节序和网络字节序 ###
主机字节序：不同的CPU有不同的字节序，这些字节序决定了数据在内存中的保存顺序，这个就是主机字节序，主机字节序最常见的有两种，即上面说到的：大端模式和小端模式。  
网络字节序：网络字节序是TCP/IP中规定的一种数据表示格式，它与具体的CPU类型、操作系统无关，从而可以保证数据在不同主机之间传输时能被正确解释。网络字节序采用大端模式。  
Linux下进行网络编程时，经常用到htons和htonl两个函数，它们就是将主机字节序转换成网络字节序。


