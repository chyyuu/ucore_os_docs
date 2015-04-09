
##### GCC基本内联汇编

GCC 提供了两内内联汇编语句（inline asm statements）：基本内联汇编语句（basic inline asm statement)和扩展内联汇编语句（extended inline asm statement）。GCC基本内联汇编很简单，一般是按照下面的格式：
	
        asm("statements");

例如：

        asm("nop"); asm("cli");
 
"asm" 和 "\_\_asm\_\_" 的含义是完全一样的。如果有多行汇编，则每一行都要加上 "\n\t"。其中的 “\n” 是换行符，"\t” 是 tab 符，在每条命令的 结束加这两个符号，是为了让 gcc 把内联汇编代码翻译成一般的汇编代码时能够保证换行和留有一定的空格。对于基本asm语句，GCC编译出来的汇编代码就是双引号里的内容。例如：
```
        asm( "pushl %eax\n\t"
             "movl $0,%eax\n\t"
             "popl %eax"
        );
```
实际上gcc在处理汇编时，是要把asm(...)的内容"打印"到汇编文件中，所以格式控制字符是必要的。再例如：
```
	asm("movl %eax, %ebx");
	asm("xorl %ebx, %edx");
	asm("movl $0, _boo);
```
在上面的例子中，由于我们在内联汇编中改变了 edx 和 ebx 的值，但是由于 gcc 的特殊的处理方法，即先形成汇编文件，再交给 GAS 去汇编，所以 GAS 并不知道我们已经改变了 edx和 ebx 的值，如果程序的上下文需要 edx 或 ebx 作其他内存单元或变量的暂存，就会产生没有预料的多次赋值，引起严重的后果。对于变量 _boo也存在一样的问题。为了解决这个问题，就要用到扩展 GCC 内联汇编语法。

参考：
- [GCC Manual， 版本为5.0.0 pre-release,6.43节（How to Use Inline Assembly Language in C Code）](https://gcc.gnu.org/onlinedocs/gcc.pdf)
- [GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)

