# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

    **答：**支持进程：分时OS需要定时切换不同的进程，因此需要硬件支持时钟中断。

    ​	支持虚存：OS进行虚存管理，需要分页机制以及内存地址映射，因此需要硬件支持MMU（含TLB等）。

    ​	支持文件系统：需要硬件支持磁盘、磁带等存储介质，并支持对它们的访问控制。

    ​	对应地，需要提供的特权指令有：设置中断使能、设置定时中断时间、设置内存寻址模式、设置TLB内页表映射、以及控制IO的相关特权指令。

    

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意哪些方面？

    **答：**x86的实模式是最早的intel 8086的工作模式（因x86架构的向下兼容及系统启动时的底层需求，x86加电启动时的工作模式即实模式），此时的内存总线仅20 bits，仅能访问1MB的地址空间，其内存地址直接指向物理内存，内存空间是不受保护的，用户程序有可能访问/修改系统的内存，造成灾难性后果。而保护模式支持内存分页、虚拟内存机制，用户程序不能直接访问物理内存，需要经过地址映射；访问非法的内存时会引起中断，转到操作系统进一步处理；另外，保护模式支持32 bit内存总线，共4GB的内存空间。

    ​	从实模式切换到保护模式，需要建立GDT，其中每一项是一个段描述符，还需设置好CS、DS等段寄存器，使其指向GDT中对应的项；最后，设置CR0的bit为1，启动保护机制与段机制。

    

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？

    **答：**物理地址是总线上访问内存/外设的最终地址。线性地址是逻辑地址经过段机制处理后得到的地址。而逻辑地址是程序进行访存时给出的地址，是虚拟内存空间中的地址。

    它们的联系和转换关系是：程序 --给出--> **逻辑地址** --段机制--> **线性地址** --页机制--> **物理地址** --访问--> 外设

    

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

    *(略)*

    

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

​	**答：**为了节省存储空间，并使处理简便，C语言又提供了一种数据结构，称为“位域”或“位段”。所谓“位域”是把一个字节中的二进位划分为几个不同的区域，并说明每个区域的位数。每个域有一个域名，允许在程序中按域名进行操作。这样就可以把几个不同的对象用一个字节的二进制位域来表示。

​	冒号后的数字即为该域的二进制长度。



- 对于如下的代码段，

```c
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```c
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

**答：** *（本题代码片段不完整，`STS_TG32`与`STS_IG32`的定义在`lab0_ex3.c`中）*

intr为32位宽，而struct gatedesc为64位宽，（宏中也没有类型转换），该段代码无法正常运行，修改如下：

```c
unsigned intr;
struct gatedesc gintr;
intr=8;
gintr=*((struct gatedesc *)&intr);
SETGATE(gintr, 0,1,2,3);
intr=*(unsigned *)&(gintr);
```

经`SETGATE`处理后，`gintr`为`0xEE00 0001 0002`，而`intr`则为`0x00010002`。



### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。


## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
