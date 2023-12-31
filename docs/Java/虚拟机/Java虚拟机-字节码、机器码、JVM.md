---
layout: default
title: Java虚拟机-字节码、机器码、JVM
parent: 虚拟机
grand_parent: Java
---

### 1. Java虚拟机（JVM）

JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

Java语言的一个非常重要的特点就是与

### 2. 机器码和字节码

首先，我们知道一段程序要想在电脑上运行，必须“翻译”成电脑能够听懂的，由0、1组成的二进制代码，这种类型的代码即称为机器码，机器码时计算机可以直接执行的、速度最快的代码。在Java中，编写好的程序即通常的.java文件需要经过

### 3. 编译器和解释器

#### 编译器

编译时从源代码（通常为高级语言）到能直接被计算机或虚拟机执行的目标代码（通常为低级语言或机器语言）的翻译过程。

#### 解释器

将相对高级的程序代码解释成电脑可以直接运行的机器码。

以Java为例，电脑是不能直接执行Java程序的，一个.java程序要想被执行，首先需要编译器将高级的.java程序文件编译成.class字节码片段，字节码经过JVM（解释器）的处理后生成可以直接执行的机器码，至此Java程序才能得以正确运行。