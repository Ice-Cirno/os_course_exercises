# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- 请描述在“计算机组成原理课”上，同学们做的MIPS CPU是从按复位键开始到可以接收按键输入之间的启动过程。

    **答：** 复位键弹起后的下一个时钟脉冲，CPU的PC指向一个固定的地址（如0xbfc00000），这个地址被映射到片内的ROM，ROM中存放着bootloader，于是控制权便交给了bootloader。Bootloader会将存放于Flash中的uCore加载到内存中，之后将控制权交给uCore（即jmp 0x80000000），uCore启动。这时uCore处于核心态，中断仍然保持屏蔽状态。uCore完成相关的自检后，启动用户程序`sh`，进入用户态，中断使能打开，uCore便可以接收按键输入了。 *（挑战性大实验）*

    

- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

    **答：** 是512 bytes的bootloader（加载程序）。因为现代OS通常远大于一个扇区的大小，磁盘上的文件系统也繁多复杂，BIOS程序无法识别所有文件系统，也就无法找到OS存放的位置。为简化BIOS，也为了让用户能够使用各种各样的OS，约定磁盘第一个扇区为主引导记录（MBR），BIOS加载bootloader，由bootloader去完成进一步的操作。

    

- 比较UEFI和BIOS的区别。

    **答：** UEFI是统一可扩展固件接口（Unified Extensible Firmware Interface）的简称，是一种用来定义操作系统与系统固件之间的软件界面的个人计算机系统规格。可扩展固件接口负责加电自检（POST）、联系操作系统以及提供连接操作系统与硬件的接口，用于替代传统的BIOS启动。[^1](https://zh.wikipedia.org/wiki/UEFI)

    比起BIOS启动，UEFI的优势主要有3点：安全性更强（但也有恶意程序成功对UEFI进行了攻击，如[rootkit](https://zh.wikipedia.org/wiki/Rootkit)）、启动配置更灵活、支持的磁盘容量更大。另外，UEFI主要用c/c++等高级语言编写，比起使用16位汇编编写的BIOS，UEFI开发更迅速、更易在不同的架构平台上移植、可以突破16位寻址能力的限制实现更多的功能，对外设的驱动程序的编写也更加友好。

    

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

    **答：** 0x55AA

    

- x86中在UEFI中的可信启动有什么作用？

    **答：** 确保系统引导的安全性，这一点是通过数字签名实现的。

    

- RV中BBL的启动过程大致包括哪些内容？

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

    **答：** 中断是来自外部IO设备的处理请求；异常是程序指令执行出现意外情况（如非法指令、溢出、除0等）的处理请求；系统调用是用户程序主动向操作系统发出的服务请求。

    

- 中断、异常和系统调用的处理流程有什么异同？

    **答：** 处理方式本质上没有差别，都是通过异常处理向量进入相应的服务例程，切换到内核态。但它们的来源和引发方式不同。中断来自外部IO设备，是异步的；异常和系统调用来自内部程序，异常是同步的，系统调用可以同步也可以异步。

    

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

    **答：** 进程管理：fork / exit / wait / exec / yield / kill / getpid / sleep

    ​	文件管理：open / close / read / write / seek / fstat / fsync / getcwd / getdirentry / dup

    ​	页面管理：pgdir

    ​	外设管理：putc

    

## 3.4 linux系统调用分析

- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

    **答：** 

    |             |                         系统调用                         |                   函数调用                    |
    | ----------: | :------------------------------------------------------: | :-------------------------------------------: |
    | CPU运行状态 |                        转为核心态                        |                    无转换                     |
    |    汇编指令 |  通过int和iret（MIPS为syscall和eret，RV为ecall和eret）   | 通过call和ret（MIPS为jal和jr，RV为jal和jalr） |
    |        性能 | 需要切换特权级、保存运行状态，可能需要切换堆栈，效率不高 | 不需切换，效率更高；但内联编译会消耗更多空间  |

    

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

    **答：** x86的函数调用，调用者负责将参数自右向左依次入栈，call指令会将返回地址压入栈，栈平衡由调用者负责维护，ebp由被调用者维护。

    ​	`int`：int指向IDT中的一项，在保护模式下，IDT表项由一个8字节长的描述符表示，int的行为与call类似，但还会将flags寄存器压入栈中；在实模式下，IDT表项为32位长的指针，int会依次压入flags、CS和返回IP，然后跳转到表项描述的地址处。（IDT的首地址均由寄存器IDTR定义）

    ​	`iret`：实模式下，iret弹出CS和flags，并跳转回被中断的程序。保护模式下，iret的行为取决于nested task (NT) flag。若NT为0，iret跳回被中断的程序，如果跳回的程序特权级比当前低（这一点可通过跳回的CS的RPL位判断），它还会从栈中弹出SS和ESP寄存器；若NT为1，iret将会做调用当前代码段的int或call的逆操作，当前task的task segment将会被更新，若该task是重复进入(re-entered)的，iret将不进行跳转，继续执行下一条指令。[^1](https://docs.oracle.com/cd/E19455-01/806-3773/instructionset-75/index.html)

    ​	`call`：将返回地址（即call的下一条指令的地址）压入栈中，并跳转到子程序。

    ​	`ret`：从栈中弹出返回地址，并跳转到该地址。

    

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

## 课堂实践 （在课堂上根据老师安排完成，课后不用做）

### 练习一

通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二

通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
