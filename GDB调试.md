## Using the GNU Debugger



找了一个视频来入门：[Linux 使用gdb调试入门。](https://www.bilibili.com/video/BV1Kq4y1D7n2?p=1)

对于 C 而言，就是你在编译时 `gcc -g -o a.out` ，这个 `-g` 保留了用于 debug 的 Info 。然后你就可以 `gdb a.out` 了。

有很多很多命令：

```bash
b main 给 main 函数打断点
b /home/a.out:11 给第 11 行打断点
r 运行
n （next） 不进函数体的单步执行
s或si （single instruction）进函数体的单步执行
c (continue)继续执行直到下一个断点或结束
k (kill)结束debug
info b 查看断点
d 5 删除编号为 5 的断点
watch i 监控 i 变量的变化（值变化时会中断程序）
info r 看寄存器的值
p i 查看变量 i 的值
p/x i 查看变量 i 的16进制
layout src 产生一个视窗（用Ctrl+x a退出）
layout asm 产生一个视窗看汇编（用Ctrl+x a退出）
si 是汇编的步进
```

启动GDB调试

gdb

退出GDB调试

quit

\#或者

exit

\#或者

Ctrl-d