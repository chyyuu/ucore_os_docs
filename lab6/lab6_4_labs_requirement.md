
## 4 实验报告要求 

从网站上下载lab6.zip后，解压得到本文档和代码目录lab6，完成实验中的各个练习。完成代码编写并检查无误后，在对应目录下执行 make handin 任务，即会自动生成lab6-handin.tar.gz。最后请一定提前或按时提交到网络学堂上。

注意有“LAB6”的注释，主要是修改default\_sched\_swide\_c中的内容。代码中所有需要完成的地方（challenge除外）都有“LAB6”和“YOUR CODE”的注释，请在提交时特别注意保持注释，并将“YOUR CODE”替换为自己的学号，并且将所有标有对应注释的部分填上正确的代码。

## 附录：执行 priority大致的显示输出 

```
$ make run-priority
......
check_swap() succeeded!
++ setup timer interrupts
kernel_execve: pid = 2, name = "priority".
main: fork ok,now need to wait pids.
child pid 7, acc 2492000, time 2001
child pid 6, acc 1944000, time 2001
child pid 4, acc 960000, time 2002
child pid 5, acc 1488000, time 2003
child pid 3, acc 540000, time 2004
main: pid 3, acc 540000, time 2004
main: pid 4, acc 960000, time 2004
main: pid 5, acc 1488000, time 2004
main: pid 6, acc 1944000, time 2004
main: pid 7, acc 2492000, time 2004
main: wait pids over
stride sched correct result: 1 2 3 4 5
all user-mode processes have quit.
init check memory pass.
kernel panic at kern/process/proc.c:426:
    initproc exit.

Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
K> 
```
