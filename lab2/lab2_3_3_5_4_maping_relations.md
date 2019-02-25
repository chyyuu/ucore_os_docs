### 系统执行中地址映射的三个阶段
原理课上讲到了页映射，段映射，以及段页式映射关系，但对如何建立段页式映射关系没有详说。其实，在lab1和lab2中都会涉及如何建立映射关系的操作。在lab1中，我们已经碰到到了简单的段映射，即对等映射关系，保证了物理地址和虚拟地址相等，也就是通过建立全局段描述符表，让每个段的基址为0，从而确定了对等映射关系。在lab2中，由于在段地址映射的基础上进一步引入了页地址映射，形成了组合式的段页式地址映射。这种方式虽然更加灵活了，但实现稍微复杂了一些。在lab2中，为了建立正确的地址映射关系，ld在链接阶段生成了ucore OS执行代码的虚拟地址，而bootloader与ucore OS协同工作，通过在运行时对地址映射的一系列“腾挪转移”，从计算机加电，启动段式管理机制，启动段页式管理机制，在段页式管理机制下运行这整个过程中，虚拟地址到物理地址的映射产生了多次变化，实现了最终的段页式映射关系：
``` 
 virt addr = linear addr = phy addr + 0xC0000000
```

下面，我们来看看这是如何一步一步实现的。观察一下链接脚本，即tools/kernel.ld文件在lab1和lab2中的区别。在lab1中：
```
ENTRY(kern_init)

SECTIONS {
            /* Load the kernel at this address: "." means the current address */
            . = 0x100000;

            .text : {
                       *(.text .stub .text.* .gnu.linkonce.t.*)
            }
```
这意味着在lab1中通过ld工具形成的ucore的起始虚拟地址从0x100000开始，注意：这个地址是虚拟地址。但由于lab1中建立的段地址映射关系为对等关系，所以ucore的物理地址也是从0x100000开始，而ucore的入口函数kern\_init的起始地址。所以在lab1中虚拟地址、线性地址以及物理地址之间的映射关系如下：
```
 lab1: virt addr = linear addr = phy addr
```

在lab2中：
```
ENTRY(kern_entry)

SECTIONS {
            /* Load the kernel at this address: "." means the current address */
            . = 0xC0100000;

            .text : {
                        *(.text .stub .text.* .gnu.linkonce.t.*)
            }
```
这意味着lab2中通过ld工具形成的ucore的起始虚拟地址从0xC0100000开始，注意：这个地址也是虚拟地址。入口函数为kern\_entry函数（在kern/init/entry.S中）。这与lab1有很大差别。但其实在lab1和lab2中，bootloader把ucore都放在了起始物理地址为0x100000的物理内存空间。这实际上说明了ucore在lab1和lab2中采用的地址映射不同。lab2在不同阶段有不同的虚拟地址、线性地址以及物理地址之间的映射关系。

也请注意，这个起始虚拟地址的变化其实并不会影响一般的跳转和函数调用，因为它们实际上是相对跳转。但是，对于绝对寻址的全局变量的引用，就需要用REALLOC宏进行一些运算来确保地址是正确的。注意到这一点可能有助于您理解下面几个阶段的某些代码，以及理解为什么这样做不会出错。

**第一个阶段**（开启保护模式，创建启动段表）是bootloader阶段，即从bootloader的start函数（在boot/bootasm.S中）到执行ucore kernel的kern\_entry函数之前，其虚拟地址、线性地址以及物理地址之间的映射关系与lab1的一样，即：

```
 lab2 stage 1: virt addr = linear addr = phy addr
```

**第二个阶段**（创建初始页目录表，开启分页模式）从kern\_entry函数开始，到pmm_init函数被执行之前。

这一阶段启动了页映射机制，启动后虚拟地址、线性地址以及物理地址之间的临时映射关系为：

```
 lab2 stage 2 before:
     virt addr = linear addr = phy addr # 线性地址在0~4MB之内三者的映射关系
     virt addr = linear addr = phy addr + 0xC0000000 # 线性地址在0xC0000000~0xC0000000+4MB之内三者的映射关系
```

可以看到，其实仅仅比第一个阶段增加了下面一行的0xC0000000偏移的映射。

然后，使用了一个绝对跳转来使内核跳转到高虚拟地址（代码在kern/init/entry.S中）：

```asm
    # update eip
    # now, eip = 0x1.....
    leal next, %eax
    # set eip = KERNBASE + 0x1.....
    jmp *%eax
next:
```

跳转完毕后，通过把boot\_pgdir[0]对应的第一个页目录表项（0\~4MB）清零来取消了临时的页映射关系：

```asm
    # unmap va 0 ~ 4M, it's temporary mapping
    xorl %eax, %eax
    movl %eax, __boot_pgdir
```

最终，离开这个阶段时，虚拟地址、线性地址以及物理地址之间的映射关系为：

```
 lab2 stage 2: virt addr = linear addr = phy addr + 0xC0000000 # 线性地址在0~4MB之内三者的映射关系
```

总结来看，这一阶段的目的就是更新映射关系的同时将运行中的内核（EIP）从低虚拟地址“迁移”到高虚拟地址，而不造成伤害。

不过，这还不是我们期望的映射关系，因为它仅仅映射了0~4MB。对于段表而言，也缺少了很多运行ucore所需的表项。

**第三个阶段**（完善段表和页表）从pmm_init函数被调用开始。pmm_init函数将页目录表项补充完成（从0~4M扩充到0~KMEMSIZE），然后更新了段映射。

这时形成了我们期望的虚拟地址、线性地址以及物理地址之间的映射关系：

```
 lab2 stage 3: virt addr = linear addr = phy addr + 0xC0000000
```

段表相应表项和TSS也被设置妥当。