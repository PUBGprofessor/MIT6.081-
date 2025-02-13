## 0、



## 一、计算机系统漫游

![image-20240816095815176](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816095815176.png)

![image-20240816103022508](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816103022508.png)

![image-20240816105602817](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816105602817.png)

## 二 、ISA介绍

![image-20240816114740109](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816114740109.png)

![image-20240816132222812](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816132222812.png)

![image-20240816132515384](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816132515384.png)

![image-20240816134911774](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816134911774.png)![image-20240816160135105](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816160135105.png)

![image-20240816160537362](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816160537362.png)![image-20240816161214986](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816161214986.png)![image-20240816163902861](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816163902861.png)![image-20240816164157356](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816164157356.png)![image-20240816164415373](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816164415373.png)

## 三、编译与链接

### 一）GCC介绍

![image-20240816172452825](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816172452825.png)

![image-20240816173057235](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816173057235.png)

![image-20240816173432726](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816173432726.png)

### 二）ELF文件（？）

![image-20240816173619417](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816173619417.png)![image-20240816173808611](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816173808611.png)![image-20240816174043671](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816174043671.png)

![image-20240816175453322](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816175453322.png)

## 四、嵌入式开发

### 一）什么是嵌入式开发

![image-20240816183906876](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816183906876.png)



### 二）交叉编译![image-20240820232438541](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240820232438541.png)

![image-20240816184607515](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816184607515.png)

### 三）GDB调试

![image-20240816184748714](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816184748714.png)

### 四）QEMU模拟

![image-20240816185125325](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816185125325.png)

### 五）项目构造工具Make

![image-20240816185601868](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816185601868.png)

![image-20240816185626551](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240816185626551.png)

## 五、RISC-V汇编语言编程

在 64 位 RISC-V 处理器中，寄存器分为通用寄存器和特权寄存器。以下是各类寄存器的名称及其作用：

### 通用寄存器

RISC-V 具有 32 个通用寄存器，命名为 `x0` 到 `x31`，每个寄存器的作用如下：

- **`x0`**: 始终为零寄存器，任何写入都将被忽略。用于提供常量零值。
- **`x1` (`ra`)**: 返回地址寄存器，存储函数调用的返回地址。
- **`x2` (`sp`)**: 栈指针，指向当前栈的顶部。
- **`x3` (`gp`)**: 全局指针，用于访问全局变量。
- **`x4` (`tp`)**: 线程指针，用于多线程环境中的线程局部存储。
- **`x5` 到 `x7` (`t0` 到 `t2`)**: 临时寄存器，调用约定中不保留的寄存器，适合临时存储。
- **`x8` (`s0`/`fp`)**: 保存寄存器，调用约定中保留的寄存器，通常用于存储帧指针。
- **`x9` (`s1`)**: 保存寄存器，调用约定中保留的寄存器。
- **`x10` 到 `x17` (`a0` 到 `a7`)**: 参数寄存器，用于函数调用时传递参数和返回值。
- **`x18` 到 `x27` (`s2` 到 `s11`)**: 保存寄存器，调用约定中保留的寄存器。
- **`x28` 到 `x31` (`t3` 到 `t6`)**: 临时寄存器，不保留的寄存器，适合临时存储。

### 特权寄存器

特权寄存器用于系统管理和控制，包括：

- **`mstatus`**: 机器状态寄存器，保存机器模式的状态信息，如中断使能位和当前特权级别。
- **`mie`**: 机器中断使能寄存器，控制中断的使能状态。
- **`mtvec`**: 机器异常向量寄存器，指定异常处理程序的入口地址。
- **`mepc`**: 机器异常程序计数器，保存异常发生时的程序计数器值。
- **`mcause`**: 机器原因寄存器，指示异常或中断的原因。
- **`mscratch`**: 机器临时寄存器，通常用于异常处理的临时数据存储。
- **`satp`**: 监督地址转换和保护寄存器，指向当前的页表。

### 小结

RISC-V 64 位处理器中的寄存器分为通用寄存器和特权寄存器，每个寄存器有特定的用途，支持程序执行、函数调用、状态保存和中断处理等多种功能。这种设计为高效的计算和系统管理提供了基础。

### 一）入门

![image-20240818170548994](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818170548994.png)

![image-20240818171406432](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818171406432.png)

### 二）指令总览

![image-20240818172755308](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818172755308.png)![image-20240818173001444](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818173001444.png)

![image-20240818173912792](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818173912792.png)![image-20240818174020503](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818174020503.png)![image-20240818174338388](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818174338388.png)![image-20240818174441212](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818174441212.png)

### 三）指令详解

![image-20240818175719049](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240818175719049.png)

![image-20240822104404304](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822104404304.png)

![image-20240822104608354](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822104608354.png)

![image-20240822105016316](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822105016316.png)

![image-20240822105107024](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822105107024.png)

 ![image-20240822105858732](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822105858732.png)

![image-20240822110141539](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822110141539.png)

![image-20240822110959090](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822110959090.png)

![image-20240822111111127](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822111111127.png)

![](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822225619227.png)

![image-20240822225927746](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822225927746.png)



![image-20240822230126089](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822230209880.png)

![image-20240822230224074](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822230224074.png)

![](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822231741916.png)

![image-20240822232217238](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822232217238.png)

![image-20240822233343546](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822233343546.png)![image-20240822233852689](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822233852689.png)

![image-20240822235127885](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822235127885.png)![image-20240822235432752](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240822235432752.png)

![image-20240823000452471](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823000452471.png)

![image-20240823000548720](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823000548720.png)![image-20240823000713489](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823000713489.png)

### 四）汇编函数调用约定

![image-20240823115451451](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823115451451.png)

![image-20240823115709015](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823115709015.png)

![image-20240823115733743](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823115733743.png)

![image-20240823124545364](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823124545364.png)

![image-20240823124807093](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823124807093.png)

### 五）汇编与C混合编程’

![image-20240823141524500](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823141524500.png)

![image-20240823142113474](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240823142113474.png)