
##### 2.4.4.2 ucore 代码编译

(1) 编译过程：在解压缩后的 ucore 源码包中使用 make 命令即可。例如 lab1中：

	chy@laptop: ~/lab1$  make

生成目标文件为 ucore.img。

(2) 保存修改：

使用 diff 命令对修改后的 ucore 代码和 ucore 源码进行比较，比较之前建议使用 make clean 命令清除不必要文件。(如果有ctags 文件，需要手工清除。)

(3)应用修改：参见 patch 命令说明。
 