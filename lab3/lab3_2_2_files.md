
### 2.2 项目组成

表1：实验三文件列表  

```
|-- boot    
|-- kern    
| |-- driver   
| | |-- …    
| | |-- ide.c    
| | \`-- ide.h    
| |-- fs    
| | |-- fs.h    
| | |-- swapfs.c    
| | \`-- swapfs.h    
| |-- init    
| | |-- …    
| | \`-- init.c    
| |-- mm    
| | |-- default\_pmm.c    
| | |-- default\_pmm.h    
| | |-- memlayout.h    
| | |-- mmu.h    
| | |-- pmm.c    
| | |-- pmm.h     
| | |-- swap.c    
| | |-- swap.h    
| | |-- swap\_fifo.c    
| | |-- swap\_fifo.h    
| | |-- vmm.c    
| | \`-- vmm.h    
| |-- sync    
| \`-- trap    
| |-- trap.c    
| \`-- …    
|-- libs     
| |-- list.h    
| \`-- …    
\`-- tools   
``` 

相对与实验二，实验三主要增加的文件如上表红色部分所示，主要修改的文件如上表紫色部分所示，其他需要用到的重要文件用黑色表示。主要改动如下：

* kern/mm/default\_pmm.[ch]：实现基于struct pmm\_manager类框架的Fist-Fit物理内存分配参考实现（分配最小单位为页，即4096字节），相关分配页和释放页等实现会间接被kmalloc/kfree等函数使用。
* kern/mm/pmm.[ch]：pmm.h定义物理内存分配类框架struct pmm\_manager。pmm.c包含了对此物理内存分配类框架的访问，以及与建立、修改、访问页表相关的各种函数实现。在本实验中会用到kmalloc/kfree等函数。
* libs/list.h：定义了通用双向链表结构以及相关的查找、插入等基本操作，这是建立基于链表方法的物理内存管理（以及其他内核功能）的基础。在lab0文档中有相关描述。其他有类似双向链表需求的内核功能模块可直接使用list.h中定义的函数。在本实验中会多次用到插入，删除等操作函数。
* kern/driver/ide.[ch]：定义和实现了内存页swap机制所需的磁盘扇区的读写操作支持；在本实验中会涉及通过swapfs\_\*函数间接使用文件中的函数。故了解即可。
* kern/fs/\*：定义和实现了内存页swap机制所需从磁盘读数据到内存页和写内存数据到磁盘上去的函数 swapfs\_read/swapfs\_write。在本实验中会涉及使用这两个函数。
* kern/mm/memlayout.h：修改了struct Page，增加了两项pra\_\*成员结构，其中pra\_page\_link可以用来建立描述各个页访问情况（比如根据访问先后）的链表。在本实验中会涉及使用这两个成员结构，以及le2page等宏。
* kern/mm/vmm.[ch]：vmm.h描述了mm\_struct，vma\_struct等表述可访问的虚存地址访问的一些信息，下面会进一步详细讲解。vmm.c涉及mm,vma结构数据的创建/销毁/查找/插入等函数，这些函数在check\_vma、check\_vmm等中被使用，理解即可。而page
fault处理相关的do\_pgfault函数是本次实验需要涉及完成的。
* kern/mm/swap.[ch]：定义了实现页替换算法的类框架struct swap\_manager。swap.c包含了对此页替换算法类框架的初始化、页换入/换出等各种函数实现。重点是要理解何时调用swap\_out和swap\_in函数。和如何在此框架下连接具体的页替换算法实现。check\_swap函数以及被此函数调用的\_fifo\_check\_swap函数完成了对本次实验中的练习2：FIFO页替换算法基本正确性的检查，可了解，便于知道为何产生错误。
* kern/mm/swap\_fifo.[ch]：FIFO页替换算法的基于类框架struct swap\_manager的简化实现，主要被swap.c的相关函数调用。重点是\_fifo\_map\_swappable函数（可用于建立页访问属性和关系，比如访问时间的先后顺序）和\_fifo\_swap\_out\_victim函数（可用于实现挑选出要换出的页），当然换出哪个页需要借助于fifo\_map\_swappable函数建立的某种属性关系，已选出合适的页。
* kern/mm/mmu.h：其中定义额也页表项的各种属性位，比如PTE\_P\\PET\_D\\PET\_A等，对于实现扩展实验的clock算法会有帮助。

本次实验的主要练习集中在vmm.c中的do\_pgfault函数和swap\_fifo.c中的\_fifo\_map\_swappable函数、\_fifo\_swap\_out\_victim函数。

#### 编译执行

编译并运行代码的命令如下：

```
make   
make qemu   
```

则可以得到如附录所示的显示内容（仅供参考，不是标准答案输出）
