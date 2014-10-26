
### 练习

#### 练习0：填写已有实验

本实验依赖实验1/2。请把你做的实验1/2的代码填入本实验中代码中有“LAB1”,“LAB2”的注释相应部分。

#### 练习1：给未被映射的地址映射上物理页（需要编程）

完成do\_pgfault（mm/vmm.c）函数，给未被映射的地址映射上物理页。设置访问权限
的时候需要参考页面所在 VMA
的权限，同时需要注意映射物理页时需要操作内存控制
结构所指定的页表，而不是内核的页表。注意：在LAB2 EXERCISE
1处填写代码。执行
```
make　qemu
```
后，如果通过check\_pgfault函数的测试后，会有“check\_pgfault()
succeeded!”的输出，表示练习1基本正确。

#### 练习2：补充完成基于FIFO的页面替换算法（需要编程）

完成vmm.c中的do\_pgfault函数，并且在实现FIFO算法的swap\_fifo.c中完成map\_swappable和swap\_out\_vistim函数。通过对swap的测试。注意：在LAB2
EXERCISE 2处填写代码。执行
```
make　qemu
```
后，如果通过check\_swap函数的测试后，会有“check\_swap()
succeeded!”的输出，表示练习2基本正确。

#### 扩展练习 Challenge：实现识别dirty bit的 extended clock页替换算法（需要编程）

challenge部分不是必做部分，不过在正确最后会酌情加分。需写出有详细的设计、分析和测试的实验报告。完成出色的可获得适当加分。
