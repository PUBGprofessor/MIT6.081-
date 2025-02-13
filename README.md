## 0、

![image-20241001154424785](source\image-20241001154424785.png)

### 通用寄存器（General-Purpose Registers）

![image-20241001154931478](source\image-20241001154931478.png)

### 控制和状态寄存器（Control and Status Registers, CSRs）

RISC-V 架构包含多个特殊寄存器，用于控制处理器状态、异常处理和中断管理。这些寄存器在 **xv6** 内核中用于管理特权级别、异常处理、内存管理等。

#### 主要 CSRs 及其用途：

| **寄存器** | **用途**                                                   |
| ---------- | ---------------------------------------------------------- |
| mstatus    | 管理机器模式的状态，如特权级别、中断使能等。               |
| mepc       | 存储发生异常时的程序计数器（PC）值，用于 `mret` 指令返回。 |
| mcause     | 存储异常的原因码。                                         |
| mtvec      | 存储机器模式的陷阱向量地址，用于异常和中断处理。           |
| medeleg    | 委派特定的异常到更低的特权级别（如监督模式）。             |
| mideleg    | 委派特定的中断到更低的特权级别。                           |
| mie        | 管理机器模式下的中断使能。                                 |
| satp       | 管理虚拟内存分页的状态，包括页表基地址和模式。             |
| pmpaddr0   | 物理内存保护（PMP）的地址寄存器 0。                        |
| pmpcfg0    | PMP 配置寄存器 0。                                         |
| ...        | 其他 PMP 地址和配置寄存器，用于细粒度内存保护。            |

###  特殊寄存器（Special Registers）

除了通用寄存器和 CSRs，RISC-V 还包含一些特殊寄存器，用于特定的操作和控制。这些寄存器在 **xv6** 中也有其特定的用途。

1. **sstatus**（Supervisor Status Register）

- **用途**：管理监督模式的状态，包括中断使能、虚拟内存使能等。
- **在 xv6 中**：主要用于配置监督模式下的特权级别和中断管理。

2. **sepc**（Supervisor Exception Program Counter）

- **用途**：存储发生异常时的程序计数器（PC）值。
- **在 xv6 中**：与 `mepc` 类似，用于在 `sret` 指令执行时返回到异常发生前的指令。

3. **sie（Supervisor Interrupt Enable Register）**

- **用途**：控制监督模式下的中断使能。
- **在 xv6 中**：用于启用和禁用**特定类型的中断**，如外部中断、软件中断和定时器中断。

4. **stvec**（Supervisor Trap-Vector Base-Address Register）

- **用途**：存储监督模式或用户模式下的**陷阱向量地址**。
- **在 xv6 中**：指向监督模式或用户模式的**异常和中断处理程序的入口地址**uservec或kernelvec。

​	5.**`sscratch`（Supervisor Scratch Register）**

​	**功能**：提供一个**临时存储区**，用于在陷阱发生时**保存和交换寄存器的值**，特别是在从用户态切换到内核态的过程中

## `scause` 的典型值（RISC-V 定义）

| **`scause` 值**      | **类型** | **含义**                   |
| -------------------- | -------- | -------------------------- |
| `0x8000000000000001` | **中断** | 软件中断（来自 `sip`）     |
| `0x8000000000000005` | **中断** | 时钟中断（来自 `sip`）     |
| `0x8000000000000009` | **中断** | 外部设备中断（来自 `sip`） |
| `0x0000000000000005` | **异常** | 非对齐访问                 |
| `0x0000000000000008` | **异常** | `ecall`（系统调用）        |
| `0x000000000000000c` | **异常** | 取指 Page Fault            |
| `0x000000000000000d` | **异常** | Load Page Fault            |
| `0x000000000000000f` | **异常** | Store Page Fault           |

## 一、操作系统接口

### 一）进程与内存

| 系统调用                  | 描述                               | 具体|
| ------------------------- | ---------------------------------- | ------------------------------------------------------------ |
| fork()                    | 创建进程                           | 一个进程可以通过系统调用 `fork` 来创建一个新的进程。`fork` 创建的新进程被称为**子进程**，子进程的内存内容同创建它的进程（父进程）一样。`fork` 函数在父进程、子进程中都返回（一次调用两次返回）。对于父进程它返回子进程的 pid，对于子进程它返回 0。`fork` 会复制父进程的文件描述符和内存，所以子进程和父进程的文件描述符一模一样。虽然 `fork` 复制了文件描述符，但每一个文件当前的偏移仍然是在父子进程之间**共享**的 |
| exit()                    | 结束当前进程                       |  |
| wait()                    | 等待子进程结束                     | `wait`的参数`status`，是一种让退出的子进程以一个整数（`32bit`的数据）的格式与等待的父进程通信方式。**如果一个进程调用`fork`两次，如果它想要等两个子进程都退出，它需要调用wait两次。** 每个`wait`会在一个子进程退出时立即返回。`wait`返回了子进程的进程号。  `wait` 返回当前进程子进程的 `PID` ，并且传入一个指针，这个指针将被修改为指向 `child exit status` ，如果我们不在乎 `child exit status` 可以传入一个无所谓的 0 指针（`(int *) 0`） 初始化时，父子进程的 `memory` 与 `registers` 相同，但之后改变其中一个的变量并不影响另一个的变量 |
| kill(pid)                 | 结束 pid 所指进程                  |                                                              |
| getpid()                  | 获得当前进程 pid                   |                                                              |
| sleep(n)                  | 睡眠 n 秒                          |                                                              |
| exec(filename, *argv)     | 加载并执行一个文件                 | `系统调用 exec 将从某个文件（通常是可执行文件）里读取内存镜像，并将其替换到调用它的进程的内存空间。exec` 接受两个参数：可执行文件名和一个字符串参数数组。**`exec`系统调用会保留当前的文件描述符表单**。所以任何在`exec`系统调用之前的文件描述符，例如`0`，`1`，`2`等。它们在新的程序中表示相同的东西。 通常来说`exec`系统调用不会返回，因为`exec`会完全替换当前进程的内存，相当于当前进程不复存在了，所以`exec`系统调用已经没有地方能返回了。**因此，我们不希望 exec 直接在 shell 里执行，否则 shell 就被 exec 替代了，我们要让 exec 在 shell 子进程中进行。**`exec`系统调用从文件中读取指令，执行这些指令，然后就没有然后了。`exec`系统调用只会当出错时才会返回，`exec` 成功执行后，并不返回调用它的程序（ calling process ），而是到 ELF header 那里。 |
| sbrk(n)                   | 为进程内存空间增加 n 字节          |                                                              |
| open(filename, flags)     | 打开文件，flags 指定读/写模式      |                                                              |
| read(fd, buf, n)          | 从文件中读 n 个字节到 buf          | 从 `fd` 读最多 n 个字节（`fd` 可能没有 n 个字节），将它们拷贝到 `buf` 中，然后返回读出的字节数。 |
| write(fd, buf, n)         | 从 buf 中写 n 个字节到文件         | `write(fd, buf, n)` 写 `buf` 中的 n 个字节到 `fd` 并且返回实际写出的字节数。如果返回值小于 n 那么只可能是发生了错误。 |
| close(fd)                 | 关闭打开的 fd                      | 系统调用 `close` 会释放一个文件描述符，使得它未来可以被 `open`, `pipe`, `dup` 等调用重用。 |
| dup(fd)                   | 复制 fd                            | `dup` 复制一个已有的文件描述符，返回一个指向同一个输入/输出对象的新描述符。这两个描述符**共享**一个文件偏移 |
| pipe( p)                  | 创建管道， 并把读和写的 fd 返回到p |                                                              |
| chdir(dirname)            | 改变当前目录                       |                                                              |
| mkdir(dirname)            | 创建新的目录                       |                                                              |
| mknod(name, major, minor) | 创建设备文件                       |                                                              |
| fstat(fd, &st)            | 返回文件信息                       | fstat(fd, &st)  stat 系统调用，将 fd（文件描述符） 所指文件信息复制到 st 中 |
| stat(buf, &st) | 返回文件信息 | stat(path, &st)  stat 系统调用，将 path（路径） 所指文件信息复制到 st 中 |
| link(f1, f2)              | 创建文件，给 f1 创建一个新名字(f2)      |                                                              |
| unlink(filename)          | 删除文件                           |  |

xv6 shell 用以上调用为用户执行程序。shell 的主要结构很简单，详见 `main` 的代码（8001）。主循环通过 `getcmd` 读取命令行的输入，然后它调用 `fork` 生成一个 shell 进程的副本。父 shell 调用 `wait`，而子进程执行用户命令。

### 二）文件描述符和I/O 重定向

![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2021102001.png)

**文件描述符**是一个整数，它代表了一个进程可以读写的被内核管理的对象。进程可以通过多种方式获得一个文件描述符，如打开文件、目录、设备，或者创建一个管道（pipe），或者复制已经存在的文件描述符。简单起见，我们常常把文件描述符指向的对象称为“文件”。

Shell会确保默认情况下，当一个程序启动时，文件描述符0连接到console的输入，文件描述符1连接到了console的输出。

每个进程都有一个从0开始的文件描述符空间。按照惯例，进程从文件描述符0读入（标准输入），从文件描述符1输出（标准输出），从文件描述符2输出错误（标准错误输出）。

一个新分配的文件描述符永远都是当前进程的最小的未被使用的文件描述符。



I/O 重定向，即将关闭原来的0或1（默认输入输出的文件描述符），再重新打开一个文件，其文件描述符为最小的未分配的文件描述符（0、1）

然后调用exec（），新执行的程序的初始I/O即为刚才打开的文件。

```c
 if(pid == 0){
    close(1);
    open("output.txt", O_WRONLY | O_CREATE);

    char *argv[] = { "echo", "this", "is", "redirected", "echo", 0 };
    exec("echo", argv);
```



### 三）管道（见pipe）

![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2021102101.png)

**如果一个管道的写端一直在写，而读端的引⽤计数是否⼤于0决定管道是否会堵塞，引用计数大于0，只写不读再次调用write会导致管道堵塞；**
**如果一个管道的读端一直在读，而写端的引⽤计数是否⼤于0决定管道是否会堵塞，引用计数大于0，只读不写再次调用read会导致管道堵塞；**
**而当他们的引用计数等于0时，只写不读会导致写端的进程收到一个SIGPIPE信号，导致进程终止，只读不写会导致read返回0,就像读到⽂件末尾⼀样。**

管道是**单通道**的：

> 因为只有一个缓存，如果是双通道，那么两个进程同时向这块缓存写数据时，这样会导致数据覆盖，即一个进程的数据被另一个进程的数据覆盖．而向套接字有读写缓存，因此套接字是双通道的．

### 四）文件系统

#### 1.目录

目录是一棵树，它的根节点是一个特殊的目录 `root`。`/a/b/c` 指向一个在目录 `b` 中的文件 `c`，而 b 本身又是在目录 `a` 中的，`a` 又是处在 `root` 目录下的。不从 `/` 开始的目录表示的是相对调用进程当前目录的目录，调用进程的当前目录可以通过 `chdir` 这个系统调用进行改变。

有很多的系统调用可以创建一个新的文件或者目录：`mkdir` 创建一个新的目录，`chdir()` 改变当前的工作目录，`open` 加上 `O_CREATE` 标志打开一个新的文件，`mknod` 创建一个新的设备文件。

#### 2.数据结构inode

`inode` 是一种数据结构，用于描述文件，一个文件可以有多个名字（`links`）。

`fstat` 用于从 `inode` 中通过文件描述符提取文件信息：

```c
struct stat {
  int dev;      // File system's disk device
  uint ino;     // Inode number
  short type;   // Type of file
  short nlink;  // Number of links to file
  uint64 size;  // Size of file in bytes
}
```

文件名和这个文件本身是有很大的区别。同一个文件（称为 `inode`）可能有多个名字，称为**连接** (`links`)。系统调用 `link` 创建另一个文件系统的名称，它指向同一个 `inode`。

#### 3.link和unlink

`link` 系统调用可以增加文件名称，如下：

```bash
open("a", O_CREATE|O_WRONLY);
link("a", "b");
```

此时 `a` 与 `b` 完全等价。`nlink` 就是 2 了。

`unlink` 系统调用可以释放名称（删除文件）。如下。

```c
unlink("a");
```

当一个 `inode` 的 `link` 都没了，并且没有文件描述符指向它了，这个 `inode` 就会被释放。如下。

```c
fd = open("/tmp/xyz", O_CREATE|O_RDWR);
unlink("/tmp/xyz");
```

上面是一种常用的创建临时 `inode` 的方法，当进程结束或者 `fd` 被关闭时，这个 `inode` 也被释放。

 `rm` 是用 `unlink` 做的（？）



v6 关于文件系统的操作都被实现为用户程序，诸如 `mkdir`，`ln`，`rm` 等等。这种设计允许任何人都可以通过用户命令拓展 shell 。现在看起来这种设计是很显然的，但是 Unix 时代的其他系统的设计都将这样的命令内置在了 shell 中，而 shell 又是内置在内核中的。

有一个例外，那就是 `cd`，它是在 shell 中实现的（8016）。`cd` 必须改变 shell 自身的当前工作目录。如果 `cd` 作为一个普通命令执行，那么 shell 就会 `fork` 一个子进程，而子进程会运行 `cd`，`cd` 只会改变*子进程*的当前工作目录。父进程的工作目录保持原样。

### 五）shell简介

当我输入`ls`时，实际的意义是我要求`Shell`运行名为`ls`的程序，文件系统中会有一个文件名为`ls`，这个文件中包含了一些计算机指令，所以实际上，当我输入`ls`时，我是在要求`Shell`运行位于文件`ls`内的这些计算机指令。

#### 重定向IO

除了运行程序以外，`Shell`还会做一些其他的事情，比如，它允许你能`重定向IO`。比如，我输入 `ls > out` 。这里执行完成之后我们看不到任何的输出，因为输出都送到了`out`文件。现在我们知道`out`文件包含了一些数据，我们可以通过`cat`指令读取一个文件 `cat out` ，并显示文件的内容。

`grep x`会搜索输入中包含`x`的行，我可以告诉`shell`将输入重定向到文件`out`，这样我们就可以查看`out`中的`x`。

```bash
grep x < out
```



## 二、操作系统组织和系统调用

### 一）Abstracting physical resources

### 二）User mode, supervisor mode, and system calls

 An application can execute only user-mode instructions (e.g., adding numbers, etc.) and is said to be running in user space, while the software in supervisor mode can also execute privileged instructions and is said to be running in kernel space. 

The software running in kernel space (or in supervisor mode) is called the kernel.

执行用户指令时， kernel stack 是空的；通过系统调用或者中断切换到内核态时， user stack 还保存着数据。

进程的线程交替使用这两种 stack 。

如何切换呢？使用 ecall 和 sret 两个指令。

### 三）xv6内核组织（Kernel organization）

kernel文件夹下：

![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022030902.png)

模块间的接口 the inter-module interfaces 在 defs.h 。

### 四）进程概览

进程是一个抽象概念，它让一个程序可以假设它独占一台机器。进程向程序提供“看上去”私有的，其他进程无法读写的内存系统（或地址空间），以及一颗“看上去”仅执行该程序的CPU。

**一个单独的隔离就是进程。 The unit of isolation in xv6 (as in other Unix operating systems) is a process.**

xv6 使用页表（由硬件实现）来为每个进程提供其独有的地址空间。页表将*虚拟地址*（x86 指令所使用的地址）翻译（或说“映射”）为*物理地址*（处理器芯片向主存发送的地址）。![figure1-1](https://th0ar.gitbooks.io/xv6-chinese/content/pic/f1-1.png)

一个进程的虚拟地址空间如下。

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022030903.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022030903.png)

如下是用于描述进程的结构。

```c
enum procstate { UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // wait_lock must be held when using this:
  struct proc *parent;         // Parent process

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
```

### 五）xv6 内核如何启动？如何执行第一个进程？

[**asm** volatile 之 C语言嵌入式汇编](https://cloud.tencent.com/developer/article/1434865)

通电后， a boot loader which is stored in read-only memory 跑起来了。这个 boot loader 把内核读入内存。然后在 machine mode 中， CPU 执行 kernel/entry.S 的 _entry 。此时，页表硬件是不可用的，虚地址就是物理地址。

The loader loads the xv6 kernel into memory at physical address 0x80000000. The reason it places the kernel at 0x80000000 ratherthan 0x0 is because the address range 0x0:0x80000000 contains I/O devices.

**Hart** 是 RISC-V 架构中用于表示硬件线程（即 CPU 核心）的术语。

**主 Hart（ID 为 0）**：负责完成大部分内核初始化工作。

**其他 Hart（ID > 0）**：等待主 Hart 完成初始化，然后进行少量的初始化工作。

### 逐行解析：

#### Ⅰ._entry:

1. **_entry:**

   - 这是一个标签，标识程序的入口点。操作系统启动时，CPU 会跳转到这个地址开始执行代码。

2. **设置C语言的堆栈（Set up a stack for C）**

   - 这部分代码的目的是为当前Hart（硬件线程）设置一个独立的堆栈空间，以便后续的C语言代码能够正确使用堆栈。

3. **注释说明（Comments）**

   ```
   # stack0 is declared in start.c,
   # with a 4096-byte stack per CPU.
   # sp = stack0 + (hartid * 4096)
   ```

   - **stack0**：在 `start.c` 中声明的一个基地址，用于所有Hart的堆栈空间。
   - **4096-byte stack per CPU**：每个CPU（Hart）有一个4096字节（4KB）的堆栈空间。
   - **sp = stack0 + (hartid \* 4096)**：堆栈指针（sp）被设置为 `stack0` 的基地址加上当前Hart的ID乘以4096，以确保每个Hart有独立的堆栈区域。

4. **加载stack0的地址到sp（Load stack0 address into sp）**

   ```
   la sp, stack0
   ```

   - **la**（Load Address）：将 `stack0` 的地址加载到堆栈指针寄存器（sp）中。此时，sp 指向 `stack0` 的起始位置。

5. **设置偏移量（Set offset）**

   ```
   li a0, 1024*4
   ```

   - **li a0, 1024\*4**：将常数 `4096`（即4KB）加载到寄存器 `a0` 中。这里的 `1024*4` 是为了明确表示4KB的大小。

6. **读取Hart ID（Read Hart ID）**

   ```
   csrr a1, mhartid
   ```

   - **csrr a1, mhartid**：从RISC-V的控制和状态寄存器（CSR）中读取当前Hart的ID（`mhartid`），并将其存储到寄存器 `a1` 中。

7. **调整Hart ID（Adjust Hart ID）**

   ```
   addi a1, a1, 1
   ```

   - **addi a1, a1, 1**：将Hart ID加1。这可能是因为Hart ID从0开始，而堆栈分配需要从1开始偏移，确保每个Hart有独立的堆栈区域。

8. **计算堆栈偏移量（Calculate stack offset）**

   ```
   mul a0, a0, a1
   ```

   - **mul a0, a0, a1**：将4KB（4096字节）乘以调整后的Hart ID，得到当前Hart的堆栈偏移量。

9. **设置堆栈指针（Set stack pointer）**

   ```
   add sp, sp, a0
   ```

   - **add sp, sp, a0**：将计算得到的堆栈偏移量加到基地址 `stack0` 上，更新堆栈指针（sp）指向当前Hart的独立堆栈区域。

10. **跳转到start函数（Jump to start() in start.c）**

    ```
    call start
    ```

    - **call start**：调用 `start` 函数，这是内核初始化过程的起点。`start` 函数通常位于 `start.c` 中，负责进一步初始化操作系统的各个部分。

11. **spin循环（spin loop）**

    ```
    spin:
        j spin
    ```

    - **spin** 标签下的指令实现了一个无限循环。当系统进入 `spin` 标签时，CPU 将不断跳转回 `spin`，保持在这个位置，避免程序继续执行。这通常用于处理未预料到的情况，确保系统在发生严重错误时不会继续执行可能导致不稳定或不可预测行为的代码


####  Ⅱ.start():

1.**设置上一个特权模式为监督模式（Supervisor Mode）**

```
unsigned long x = r_mstatus();
x &= ~MSTATUS_MPP_MASK;
x |= MSTATUS_MPP_S;
w_mstatus(x);
```

- **`r_mstatus()`**：读取当前的 `mstatus` 控制和状态寄存器（CSR）。
- **`x &= ~MSTATUS_MPP_MASK;`**：清除 `MPP`（Machine Previous Privilege）字段，确保上一个特权级别被重置。
- **`x |= MSTATUS_MPP_S;`**：将 `MPP` 字段设置为监督模式（Supervisor Mode）。
- **`w_mstatus(x);`**：将修改后的值写回 `mstatus` 寄存器。

**目的**：确保在执行 `mret` 指令时，CPU 切换回监督模式，而不是其他模式（如用户模式）。

2. **设置异常程序计数器（Exception Program Counter）为 `main` 函数**

```
w_mepc((uint64)main);
```

- **`w_mepc((uint64)main);`**：将 `mepc` 寄存器   设置为 `main` 函数的地址。

**目的**：当 `mret` 指令执行时，CPU 将跳转到 `main()` 函数，开始执行内核的主逻辑。

3. **禁用分页**

```
w_satp(0);
```

- **`w_satp(0);`**：将 `satp` 寄存器设置为 `0`，**禁用**虚拟内存分页。

**目的**：简化内存管理，暂时禁用分页机制。内核在此阶段还未初始化完整的虚拟内存系统。

4. **委派所有中断和异常到监督模式**

```
w_medeleg(0xffff);
w_mideleg(0xffff);
w_sie(r_sie() | SIE_SEIE | SIE_STIE | SIE_SSIE);
```

- **`w_medeleg(0xffff);`**：将所有机器级异常委派给监督模式。

- **`w_mideleg(0xffff);`**：将所有机器级中断委派给监督模式。

- `w_sie(r_sie() | SIE_SEIE | SIE_STIE | SIE_SSIE);`

  ：

  - 读取当前的 `sie` 寄存器值。
  - 启用监督模式的外部中断（`SIE_SEIE`）、定时器中断（`SIE_STIE`）和软件中断（`SIE_SSIE`）。
  - 写回修改后的值到 `sie` 寄存器。

**目的**：确保中断和异常在监督模式下处理，而不是在机器模式下。

5. **配置物理内存保护（Physical Memory Protection, PMP）**

```
w_pmpaddr0(0x3fffffffffffffull);
w_pmpcfg0(0xf);
```

- **`w_pmpaddr0(0x3fffffffffffffull);`**：设置 PMP 地址寄存器 0 为最大的物理地址，覆盖整个物理内存范围。
- **`w_pmpcfg0(0xf);`**：配置 PMP 配置寄存器 0，启用读（R）、写（W）、执行（X）和权限位（A）。

**目的**：通过 PMP 机制，赋予监督模式对所有物理内存的完全访问权限，确保内核能够自由地管理内存。

6. **初始化定时器中断**

```
timerinit();
```

- **`timerinit();`**：初始化定时器，设置时钟中断的频率和行为。

**目的**：使系统能够接收定时器中断，用 于调度和时间管理。

7. **将每个 CPU 的 Hart ID 存储在其 `tp` 寄存器中，用于 `cpuid()`**

```
int id = r_mhartid();
w_tp(id);
```

- **`r_mhartid();`**：读取当前 CPU（Hart）的 ID。
- **`w_tp(id);`**：将 Hart ID 写入线程指针寄存器（`tp`）。

**目的**：在用户程序和内核中，通过 `tp` 寄存器可以获取当前执行代码的 CPU ID，用于识别和管理多核处理器环境。

8. **切换到监督模式并跳转到 `main()`**

```
asm volatile("mret");
```

- **`asm volatile("mret");`**：执行 `mret` 指令，从机器模式返回到监督模式，并跳转到 `mepc` 寄存器中设置的地址（即 `main` 函数）。

**目的**：完成 CPU 特权级别的切换，将控制权交给内核的主函数 `main()`，开始内核的进一步初始化和操作。

#### Ⅲ.main：（*）

初始化各抽象的内存控制系统

然后调用第一个用户程序：userinit()

#### Ⅳ.userinit()：

开辟一个页，创造了一个进程：exec("/init") （`initcode` 数组实际上存储的是从 `initcode.S` 中汇编而来的机器代码，用户程序的机器指令以字节形式存储，靠bit写入实现）

#### Ⅴ.重回main，调用scheduler()调度器：

1. **`struct cpu \*c = mycpu();`**：
   - 获取当前处理器的信息，`mycpu()` 是一个函数，返回当前执行的 CPU 的指针。在多核处理器系统中，每个 CPU 都有一个相关的 `struct cpu` 结构。
2. **`c->proc = 0;`**：
   - 初始化当前 CPU 处理的进程指针为 `0`，表示当前没有进程在运行。
3. **`for(;;)`**：
   - 无限循环，调度器不断运行以查找可运行的进程。
4. **`intr_on();`**：
   - 启用中断，确保设备可以产生中断，以避免死锁。这对于实现进程切换和设备 I/O 是必要的。
5. **`for(p = proc; p < &proc[NPROC]; p++)`**：
   - 遍历进程表中的所有进程（`proc` 是一个指向进程数组的指针，`NPROC` 是最大进程数）。
6. **`acquire(&p->lock);`**：
   - 获取进程 `p` 的锁，以确保对其状态的修改是线程安全的。
7. **`if(p->state == RUNNABLE)`**：
   - 检查进程 `p` 的状态是否为可运行（`RUNNABLE`）。如果是，表示该进程可以被调度执行。
8. **`p->state = RUNNING;`**：
   - 将进程状态设置为运行中（`RUNNING`）。
9. **`c->proc = p;`**：
   - 将当前 CPU 的进程指针设置为正在运行的进程 `p`。
10. **`swtch(&c->context, &p->context);`**：
    - 调用 `swtch` 函数进行上下文切换，将 CPU 的上下文切换到进程 `p` 的上下文。上下文切换是操作系统从一个进程切换到另一个进程的过程。
11. **`c->proc = 0;`**：
    - 当进程 `p` 的运行完成并返回时，设置当前 CPU 的进程指针为 `0`，表示当前没有进程在运行。
12. **`release(&p->lock);`**：
    - 释放进程 `p` 的锁，允许其他线程访问该进程。

#### Ⅵ.CPU开始循环处理各个进程

第一个就是刚创建的进程exec(“/init”)，即执行init.c这个文件

#### Ⅶ.init程序：启动shell

`init` 进程是操作系统启动后第一个用户进程，负责启动和管理其他用户进程。下面是代码的详细解释：

1. **打开控制台**:

   ```
   if(open("console", O_RDWR) < 0){
       mknod("console", CONSOLE, 0);
       open("console", O_RDWR);
   }
   dup(0);  // stdout
   dup(0);  // stderr
   ```

   - 首先尝试打开 `console` 文件，如果打开失败，则创建一个控制台设备节点并再次尝试打开。
   - `dup(0)` 被调用两次，将标准输出和标准错误输出重定向到控制台。

2. **主循环**:

   ```
   for(;;){
       printf("init: starting sh\n");
       pid = fork();
   ```

   - 无限循环中，首先输出信息，表示将要启动 shell。
   - 使用 `fork()` 创建一个新的进程。如果创建进程失败，则输出错误信息并退出。

3. **执行 shell**:

   ```
   if(pid == 0){
       exec("sh", argv);
       printf("init: exec sh failed\n");
       exit(1);
   }
   ```

   - 如果 `fork()` 返回值为 `0`，表示这是子进程。在子进程中，调用 `exec("sh", argv);` 来执行 shell 程序。如果 `exec` 失败，则输出错误信息并退出。

4. **等待 shell 进程**:

   ```
   for(;;){
       wpid = wait((int *) 0);
       if(wpid == pid){
           // the shell exited; restart it.
           break;
       } else if(wpid < 0){
           printf("init: wait returned an error\n");
           exit(1);
       } else {
           // it was a parentless process; do nothing.
       }
   }
   ```

   - 在一个无限循环中调用 `wait()`，等待子进程结束。
   - 如果返回的进程 ID `wpid` 与 `pid` 相等，表示 shell 进程退出，跳出循环以重新启动 shell。
   - 如果 `wait()` 返回负值，则输出错误信息并退出。
   - 如果返回的 `wpid` 是其他进程的 ID，说明该进程是无父进程的进程，可以忽略。

因此：

- 这段代码的核心功能是不断地启动和重启 `sh`（shell）进程，确保系统总有一个 shell 进程在运行。
- 当 shell 进程结束时，`init` 进程会重新启动它，从而保持用户可以交互的环境。

### 四）内联汇编的格式（？）







### 五）隔离性

#### 1.user/kernel mode

为了支持user/kernel mode，处理器会有两种操作模式，第一种是user mode，第二种是kernel mode。当运行在kernel mode时，CPU可以运行特定权限的指令（privileged instructions）；当运行在user mode时，CPU只能运行普通权限的指令（unprivileged instructions）。

在RISC-V中，有一个专门的指令用来实现**user mode和kernel mode的切换**，叫做ECALL。ECALL接收一个数字参数，当一个用户程序想要将程序执行的控制权转移到内核，它只需要执行ECALL指令，并传入一个数字。这里的数字参数代表了应用程序想要调用的System Call。

#### 2.page table



### 六）宏内核 vs 微内核 （Monolithic Kernel vs Micro Kernel）

宏内核 bug 容易多，但是性能好。

这里解释一下为啥微内核性能差一些。

在微内核中，对于任何文件系统的交互，都需要分别完成2次用户空间<->内核空间的跳转。与宏内核对比，在宏内核中如果一个应用程序需要与文件系统交互，只需要完成1次用户空间<->内核空间的跳转，所以微内核的的跳转是宏内核的两倍。通常微内核的挑战在于性能更差，这里有两个方面需要考虑：

- 在user/kernel mode反复跳转带来的性能损耗。
- 在一个类似宏内核的紧耦合系统，各个组成部分，例如文件系统和虚拟内存系统，可以很容易的共享page cache。而在微内核中，每个部分之间都很好的隔离开了，这种共享更难实现。进而导致更难在微内核中得到更高的性能。

### 七）编译运行kernel

#### 1.代码结构

第一个是kernel。我们可以ls kernel的内容，里面包含了基本上所有的内核文件。因为XV6是一个宏内核结构，这里所有的文件会被编译成一个叫做kernel的二进制文件，然后这个二进制文件会被运行在kernle mode中。 第二个部分是user。这基本上是运行在user mode的程序。这也是为什么一个目录称为kernel，另一个目录称为user的原因。

第三部分叫做mkfs。它会创建一个空的文件镜像，我们会将这个镜像存在磁盘上，这样我们就可以直接使用一个空的文件系统。

#### 2.简单介绍内核如何编译的

Makefile 会读取一个 C 文件，比如 proc.c ；然后调用 gcc 编译器，生成一个文件叫做 proc.s ，这是RISC-V 汇编语言文件；之后再走到汇编解释器（ assembler ），生成 proc.o ，这是汇编语言的二进制格式。

Makefile 会为所有内核文件做相同的操作，比如说 pipe.c ，会按照同样的套路，先经过 gcc 编译成 pipe.s ，再通过汇编解释器生成 pipe.o 。

之后，系统加载器（ Loader ）会收集所有的 .o 文件，将它们链接在一起，并生成内核文件。

这里生成的内核文件就是我们将会在QEMU中运行的文件。 **同时，为了你们的方便，Makefile还会创建kernel.asm，这里包含了内核的完整汇编语言，你们可以通过查看它来定位究竟是哪个指令导致了Bug。**

（ kernel.asm 有一万多行）

#### 3.QEMU

Makefile 里编译文件然后调用QEMU（qemu-system-riscv64指令）。这里本质上是通过C语言来模拟仿真RISC-V处理器。

我们来看传给QEMU的几个参数：

- `-kernel`：这里传递的是内核文件（kernel目录下的kernel文件），这是将在QEMU中运行的程序文件。
- `-m`：这里传递的是RISC-V虚拟机将会使用的内存数量
- `-smp`：这里传递的是虚拟机可以使用的CPU核数
- `-drive`：传递的是虚拟机使用的磁盘驱动，这里传入的是fs.img文件

这样，XV6系统就在QEMU中启动了。

QEMU 表现的就像一个真正的计算机一样。当你想到 QEMU 时，你不应该认为它是一个 C 程序，你应该把它想成一个真正的主板。

在内部，在QEMU的主循环中，只在做一件事情：

- 读取4字节或者8字节的RISC-V指令。
- 解析RISC-V指令，并找出对应的操作码（op code）。我们之前在看 kernel.asm 的时候，看过一些操作码的二进制版本。通过解析，或许可以知道这是一个 ADD 指令，或者是一个 SUB 指令。
- 之后，在软件中执行相应的指令。

这基本上就是QEMU的全部工作了，对于每个CPU核，QEMU都会运行这么一个循环。

## 三、页表（Page Table）

在 RISC-V 架构中，**`satp`（Supervisor Address Translation and Protection）寄存器**是一个非常重要的控制寄存器，主要用于管理虚拟内存的地址转换以及页表的保护机制。它包含了几个重要的字段，用来控制虚拟内存的启用、页表的基地址和地址转换模式等。`satp` 寄存器通常只有在**超级用户模式（Supervisor mode, S-mode）**下才能访问和操作。

### 一)Paging hardware

#### 1.`satp` 寄存器的结构（对于 RISC-V 64 位架构）

对于 **64 位 RISC-V 架构（RV64）**，`satp` 寄存器是一个 64 位的寄存器，通常由以下三个主要部分组成：

1. **模式字段（mode，最高的 4 位）**：
   - 这个字段决定了当前的地址转换模式。不同的值表示不同的页表模式，例如：
     - `0`：关闭虚拟内存（裸机模式）
     - `8`：启用 SV39 地址转换模式（39 位虚拟地址，三级页表）
     - `9`：启用 SV48 地址转换模式（48 位虚拟地址，四级页表）
     - `10`：启用 SV57 地址转换模式（57 位虚拟地址，五级页表）
2. **页表基地址字段（PPN, Page-Table Physical Page Number，中间的 44 位）**：
   - 这个字段存储页表的物理地址。由于页表的地址通常对齐到页面大小（一般是 4 KB 的页面），只需要存储物理页面号（PPN）。PPN 直接指向根页表的物理地址。
3. **ASID（Address Space Identifier，最低的 16 位）**：
   - 用于区分不同进程的地址空间。在虚拟内存管理中，多个进程可以共享同一个页表，这个字段用于标识当前页表属于哪个进程，避免 TLB（Translation Lookaside Buffer）污染。ASID 的使用可以提高上下文切换时的效率。

#### 2.地址转换

每个 PTE 有一个 44 位的物理页编号和一些标志信息。硬件（the paging number）通过 27 位的去找到 PTE ，此时页表中 PPN 就是物理地址的 44 位，另外 12 位来自于原有的 Offset 。原文： The paging hardware translates a virtual address by using the top 27 bits of the 39 bits to index into the page table to find a PTE, and making a 56-bit physical address whose top 44 bits come from the PPN in the PTE and whose bottom 12 bits are copied from the original virtual address. 如下图。

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022031401.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022031401.png)

然而，如果我们把 227 这么大的内存拿出来保存 PTE ，那将浪费好多内存空间。所以实际上我们是用三级页表，这 27 位中分别用前中后三个 9 位来索引各级页表。如下图。

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022031402.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022031402.png)

也正因如此，第一级页表大小为 4096 个字节也就是 512 个 PTE 。为啥？因为 2^9=512 。之后有几个二级页表，根据实际情况开辟多少个，这样就实现了节省空间（相比不分级的页表）。

还没有这个页表时，硬件就返回一个 page-fault exception 。

此外， PTE 的标志位如下图所示。

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022031403.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022031403.png)

#### 

### 二）Kernel address space

#### 1.kernel的虚拟地址和物理地址转换

![image-20240930222608595](C:\Users\刘佳豪\AppData\Roaming\Typora\typora-user-images\image-20240930222608595.png)

图中的右半部分的结构完全由硬件设计者决定。如你们上节课看到的一样，当操作系统启动时，会从地址`0x80000000`开始运行，这个地址其实也是由硬件设计者决定的。具体的来说，如果你们看一个主板如下：

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022031505.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022031505.png)

中间是RISC-V处理器，我们现在知道了处理器中有4个核，每个核都有自己的MMU和TLB。处理器旁边就是DRAM芯片。

主板的设计人员决定了，在完成了虚拟到物理地址的翻译之后，如果得到的物理地址大于0x80000000会走向DRAM芯片，如果得到的物理地址低于0x80000000会走向不同的I/O设备。

### 三）Code: creating an address space

#### 1.vm

#### 2.Translation Look-aside Buffer (TLB)

### 四）Physical memory allocation

 xv6 uses the physical memory between the end of the kernel and PHYSTOP for run-time alloca tion.

It keeps track of which pages are free by threading a linked list through the pages themselves. Allocation consists of removing a page from the linked list; freeing consists of adding the freed page to the list.

### 五）Code: Physical memory allocator

#### 1. kalloc

### 六）Process address space

![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022031405.png)

- First, different processes’ page tables translate user addresses to different pages of physical memory, so that each process has private user memory. 进程空间隔离。
- Second, each process sees its memory as having contiguous virtual addresses starting at zero, while the process’s physical memory can be non-contiguous. 进程自己看到的地址是连续的。
- Third, the kernel maps a page with trampoline code at the top of the user address space, thus a single page of physical memory shows up in all address spaces. 用户态和内核态可以公用一些物理内存（通过 trampoline code at the top of the user address space ？）

##### 栈（stack）：



栈之所以很重要的原因是，它使得我们的函数变得有组织，且能够正常返回。

下面是一个非常简单的栈的结构图，其中每一个区域都是一个Stack Frame，每执行一次函数调用就会产生一个Stack Frame。

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022032202.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022032202.png)

**每一次我们调用一个函数，函数都会为自己创建一个Stack Frame，并且只给自己用。函数通过移动Stack Pointer来完成Stack Frame的空间分配。**

对于Stack来说，是从高地址开始向低地址使用。所以栈总是向下增长。当我们想要创建一个新的Stack Frame的时候，总是对当前的Stack Pointer做减法。

有关Stack Frame有两件事情是确定的：

- Return address总是会出现在Stack Frame的第一位
- 指向前一个Stack Frame的指针也会出现在栈中的固定位置

有关Stack Frame中有两个重要的寄存器， **第一个是SP（Stack Pointer），它指向Stack的底部并代表了当前Stack Frame的位置。第二个是FP（Frame Pointer），它指向当前Stack Frame的顶部。** 因为Return address和指向前一个Stack Frame的的指针都在当前Stack Frame的固定位置，所以可以通过当前的FP寄存器寻址到这两个数据。

##### 关于stack的gdb调试（？）

### 七）代码：sbrk和exec



## 四、Traps and system calls

打断 CPU 执行指令的三个方式：

- system call （ecall 指令）
- exception
- device interrupt

上面统称为 trap 。 **We often want traps to be transparen.** 此外，三种 trap 也可以划分成：

- traps from user space
- traps from kernel space
- traps from timer interrupts

### 一）RISC-V trap machinery



### 二）Traps from user space

![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022040401.png)

具体见book

### 三）Code: Calling system calls



### 四）Code: System call arguments

通过保存到trapframe中的寄存器的值，传递参数：p->trapframe->a0 ...... a7

通过kernel/syscall.c中的函数argint,argaddr,argstr等，和通过kernel/vm.c中的函数copyin,copyinstr等

### 五）Traps from kernel space



### 六）Page fault

#### 1.Lazy page allocation (sbrk)

核心思想非常简单，sbrk系统调基本上不做任何事情，唯一需要做的事情就是提升p->sz，将p->sz增加n，其中n是需要新分配的内存page数量。但是内核在这个时间点并不会分配任何物理内存。之后在某个时间点，应用程序使用到了新申请的那部分内存，这时会触发page fault，因为我们还没有将新的内存映射到page table。所以，如果我们解析一个大于旧的`p->sz`，但是又小于新的`p->sz`（注，也就是旧的`p->sz + n`）的虚拟地址，我们希望内核能够分配一个内存page，**并且重新执行指令**。

#### 2.Zero Fill On Demand

当编译器在生成二进制文件时，编译器会填入这三个区域。text区域是程序的指令，data区域存放的是初始化了的全局变量，BSS包含了未被初始化或者初始化为0的全局变量。**在物理内存中，我只需要分配一个page，这个page的内容全是0。然后将所有虚拟地址空间的全0的page都map到这一个物理page上。这样至少在程序启动的时候能节省大量的物理内存分配。**

#### 3.Copy On Write Fork

当我们创建子进程时，与其创建，分配并拷贝内容到新的物理内存，其实我们可以直接共享父进程的物理内存page。所以这里，我们可以设置子进程的PTE指向父进程对应的物理内存page。

对于PTE的标志位，我之前介绍过第0bit到第7bit，但是没有介绍最后两位RSW。这两位保留给supervisor software使用，supervisor softeware指的就是内核。内核可以随意使用这两个bit位。所以可以做的一件事情就是，**将bit8标识为当前是一个copy-on-write page**。

#### 4.Demand paging

程序的二进制文件可能非常的巨大，将它全部从磁盘加载到内存中将会是一个代价很高的操作。又或者data区域的大小远大于常见的场景所需要的大小，我们并不一定需要将整个二进制都加载到内存中。

如果你们再看PTE，我们有RSW位， **你们可以发现在bit7，对应的就是Dirty bit。** 当硬件向一个page写入数据，会设置dirty bit，之后操作系统就可以发现这个page曾经被写入了。类似的，还有一个Access bit，任何时候一个page被读或者被写了，这个Access bit会被设置。

#### 5.Memory Mapped Files

这节课最后要讨论的内容，也是后面的一个实验，就是memory mapped files。这里的核心思想是，将完整或者部分文件加载到内存中，这样就可以通过内存地址相关的load或者store指令来操纵文件。为了支持这个功能，一个现代的操作系统会提供一个叫做mmap的系统调用。这个系统调用会接收一个虚拟内存地址（VA），长度（len），protection，一些标志位，一个打开文件的文件描述符，和偏移量（offset）。

### 七）Real world

首先是计算机中总共有多少内存（33048332），如果你再往后看的话，你会发现大部分内存都被使用了（4214604 + 26988148）。但是大部分内存并没有被应用程序所使用，而是被buff/cache用掉了。这在一个操作系统中还挺常见的，因为我们不想让物理内存就在那闲置着，我们想让物理内存被用起来，所以这里大块的内存被用作buff/cache。可以看到还有一小块内存是空闲的（1845580），但是并不多。

## 五、Interrupt and device driver(?)

**所有的设备都连接到处理器上，处理器上是通过Platform Level Interrupt Control，简称PLIC来处理设备中断。PLIC会管理来自于外设的中断。如果我们再进一步深入的查看PLIC的结构图。**

UART设备通常包含以下关键寄存器：

- **Transmit Holding Register (THR)**：用于存储要发送的数据。
- **Receiver Buffer Register (RBR)**：用于存储接收的数据。
- **Interrupt Enable Register (IER)**：用于启用或禁用中断。
- **Line Status Register (LSR)**：用于指示UART的状态，如是否准备好发送或接收数据。

### 一）Interrupt硬件部分

通用异步收发传输器（Universal Asynchronous Receiver/Transmitter)，通常称作UART。

**所有的设备都连接到处理器上，处理器上是通过Platform Level Interrupt Control，简称PLIC来处理设备中断。PLIC会管理来自于外设的中断。如果我们再进一步深入的查看PLIC的结构图。**

[![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022041704.png)](https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/docs/drafts/images/2022041704.png)

主板可以连接以太网卡，MicroUSB，MicroSD等，主板上的各种线路将外设和CPU连接在一起。



### 二）Interrupt软件部分

在 xv6 中，中断处理程序的入口通常是由 `trap.c` 文件中的 `trap` 函数定义的。具体来说，`trap` 函数会被调用以处理所有类型的中断和异常，包括外部中断、系统调用等。

当中断发生时，处理器会根据当前的中断向量表（通常在 `stvec` 寄存器中定义）跳转到相应的处理程序。在 xv6 中，`trap` 函数会根据中断的类型（通过 `scause` 寄存器来判断）决定后续的处理逻辑。

#### 设备驱动概述（UART设备为例）

通常来说，管理设备的代码称为驱动，所有的驱动都在内核中。如果我们查看代码的结构，我们可以发现大部分驱动都分为两个部分，bottom/top。通常情况下， **驱动中会有一些队列（或者说buffer），top部分的代码会从队列中读写数据，而Interrupt handler（bottom部分）同时也会向队列中读写数据。这里的队列可以将并行运行的设备和CPU解耦开来。**

#### 如何对设备进行编程（memory mapped I/O）

通常来说，对设备编程是通过memory mapped I/O完成的。在SiFive的手册中，设备地址出现在物理地址的特定区间内，这个区间由主板制造商决定。操作系统需要知道这些设备位于物理地址空间的具体位置，然后再通过普通的load/store指令对这些地址进行编程。load/store指令实际上的工作就是读写设备的控制寄存器。例如，对网卡执行store指令时，CPU会修改网卡的某个控制寄存器，进而导致网卡发送一个packet。所以这里的load/store指令不会读写内存，而是会操作设备。并且你需要阅读设备的文档来弄清楚设备的寄存器和相应的行为，有的时候文档很清晰，有的时候文档不是那么清晰

尽管Console背后是UART设备，但是从应用程序来看，它就像是一个普通的文件。Shell程序只是向文件描述符2写了数据，它并不知道文件描述符2对应的是什么。在Unix系统中，设备是由文件表示。

### 三）详细UART驱动实现

#### 1.top部分

#### 2.bottom部分





sleep会将当前在运行的进程存放于sleep数据中。它传入的参数是需要等待的信号，在这个例子中传入的是uart_tx_r的地址。在uartstart函数中，**一旦buffer中有了空间，会调用与sleep对应的函数wakeup，传入的也是uart_tx_r的地址。任何等待在这个地址的进程都会被唤醒。有时候这种机制被称为conditional synchronization（条件信号）**

## 六、Locking

![img](https://github.com/PiperLiu/CS-courses-notes/raw/master/notes/mit6.s081/docs/drafts/images/2022042301.png)

### 1. Deadlock and lock ordering

If a code path through the kernel must hold several locks at the same time, it is important that all code paths acquire those locks in the same order. If they don’t, there is a risk of deadlock.

### 2. Re-entrant locks

For this reason, xv6 uses the simpler to understand non-re-entrant locks. As long as program mers keep the locking rules in mind, however, either approach can be made to work. If xv6 were touse re-entrant locks, one would have to modify acquire to notice that the lock is currently held by the calling thread. One would also have to add a count of nested acquires to struct spinlock, in similar style to push_off, which is discussed next.

### 3. Locks and interrupt handlers

xv6使用的方法：当正在执行的进程不持有任何锁的时候才yield

### 4. Instruction and memory ordering

```c
 // Tell the C compiler and the processor to not move loads or stores
  // past this point, to ensure that the critical section's memory
  // references happen strictly after the lock is acquired.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();
```

### 5.  Sleeplocks（important）

```c
struct sleeplock {
  uint locked;       // Is the lock held?
  struct spinlock lk; // spinlock protecting this sleep lock
  
  // For debugging:
  char *name;        // Name of lock.
  int pid;           // Process holding lock
};
```

关键函数：wakeup 、 sleep

**为什么sleeplock里面包含一个spinlock？**

答：**保护锁的元数据**（如 `locked` 和 `pid`）的修改，避免lose wakeup：某一个进程在执行sleep函数过程中，另一进程刚好释放这个sleeplock，当前一个进程进入sheduler时，该sleeplock已经不会被唤醒了。

## 七、Scheduling

### Xv6 线程切换描述

我们或许会运行多个用户空间进程，例如C compiler（CC），LS，Shell，它们或许会，也或许不会想要同时运行。在用户空间，每个进程有自己的内存，对于我们这节课来说，我们更关心的是每个进程都包含了一个用户程序栈（user stack），并且当进程运行的时候，它在RISC-V处理器中会有程序计数器和寄存器。当用户程序在运行时，实际上是用户进程中的一个用户线程在运行。如果程序执行了一个系统调用或者因为响应中断走到了内核中，那么相应的用户空间状态会被保存在程序的trapframe中，同时属于这个用户程序的内核线程被激活。所以首先，用户的程序计数器，寄存器等等被保存到了trapframe中，之后CPU被切换到内核栈上运行，实际上会走到trampoline和usertrap代码中。之后内核会运行一段时间处理系统调用或者执行中断处理程序。在处理完成之后，如果需要返回到用户空间，trapframe中保存的用户进程状态会被恢复。

当XV6从CC程序的内核线程切换到LS程序的内核线程时：

- XV6会首先会将CC程序的内核线程的内核寄存器保存在一个context对象中。
- 类似的，因为要切换到LS程序的内核线程，那么LS程序现在的状态必然是RUNABLE，表明LS程序之前运行了一半。这同时也意味着LS程序的用户空间状态已经保存在了对应的trapframe中，更重要的是，LS程序的内核线程对应的内核寄存器也已经保存在对应的context对象中。所以接下来，XV6会恢复LS程序的内核线程的context对象，也就是恢复内核线程的寄存器。
- 之后LS会继续在它的内核线程栈上，完成它的中断处理程序（注，假设之前LS程序也是通过定时器中断触发的pre-emptive scheduling进入的内核）。
- 然后通过恢复LS程序的trapframe中的用户进程状态，返回到用户空间的LS程序中。
- 最后恢复执行LS。

### Multiprocessors and locking多处理器和锁定

### Thread switching线程切换



## 八、文件系统

![image-20250213102148542](source\image-20250213102148542.png)

Github包含其他资源：https://github.com/PUBGprofessor/MIT6.081-

## 参考资料

1. 课表：https://pdos.csail.mit.edu/6.828/2021/schedule.html

2. 笔记： https://github.com/PiperLiu/CS-courses-notes/blob/master/notes/mit6.s081/

3. 其他文章：

   https://www.cnblogs.com/lilpig/p/17168211.html#ecall

   https://blog.csdn.net/fatcat123/article/details/117769246

   https://blog.csdn.net/G129558/article/details/132571881

   https://blog.csdn.net/zzy980511/article/details/130591088
