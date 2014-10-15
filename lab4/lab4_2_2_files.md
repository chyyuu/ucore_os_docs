
### 2.2 项目组成 

```
├── boot   
├── kern  
│ ├── debug  
│ ├── driver  
│ ├── fs   
│ ├── init  
│ │ ├── init.c   
│ │ └── ...  
│ ├── libs  
│ │ ├── rb\_tree.c  
│ │ ├── rb\_tree.h  
│ │ └── ...   
│ ├── mm   
│ │ ├── kmalloc.c   
│ │ ├── kmalloc.h   
│ │ ├── memlayout.h   
│ │ ├── pmm.c   
│ │ ├── pmm.h   
│ │ ├── swap.c   
│ │ ├── vmm.c   
│ │ └── ...   
│ ├── process  
│ │ ├── entry.S  
│ │ ├── proc.c   
│ │ ├── proc.h  
│ │ └── switch.S  
│ ├── schedule  
│ │ ├── sched.c  
│ │ └── sched.h  
│ ├── sync  
│ │ └── sync.h  
│ └── trap  
│ ├── trapentry.S  
│ └── ...  
├── libs  
│ ├── hash.c   
│ ├── stdlib.h  
│ ├── unistd.h   
│ └── ...  
├── Makefile  
└── tools  
```

相对与实验三，实验四主要增加的文件如上表红色部分所示，主要修改的文件如上表紫色部分所示。主要改动如下：

● kern/process/ （新增进程管理相关文件）

proc.[ch]：新增：实现进程、线程相关功能，包括：创建进程/线程，初始化进程/线程，处理进程/线程退出等功能

entry.S：新增：内核线程入口函数kernel\_thread\_entry的实现

switch.S：新增：上下文切换，利用堆栈保存、恢复进程上下文

● kern/init/

init.c：修改：完成进程系统初始化，并在内核初始化后切入idle进程

● kern/mm/
（基本上与本次实验没有太直接的联系，了解kmalloc和kfree如何使用即可）

kmalloc.[ch]：新增：定义和实现了新的kmalloc/kfree函数。具体实现是基于slab分配的简化算法
（只要求会调用这两个函数即可）

memlayout.h：增加slab物理内存分配相关的定义与宏 （可不用理会）。

pmm.[ch]：修改：在pmm.c中添加了调用kmalloc\_init函数,取消了老的kmalloc/kfree的实现；在pmm.h中取消了老的kmalloc/kfree的定义

swap.c：修改：取消了用于check的Line 185的执行

vmm.c：修改：调用新的kmalloc/kfree

● kern/trap/

trapentry.S：增加了汇编写的函数forkrets，用于do\_fork调用的返回处理。

● kern/schedule/

sched.[ch]：新增：实现FIFO策略的进程调度

● kern/libs

rb\_tree.[ch]：新增：实现红黑树，被slab分配的简化算法使用（可不用理会）

**编译执行**

编译并运行代码的命令如下：

```
make
make qemu
```

则可以得到如附录A所示的显示内容（仅供参考，不是标准答案输出）
