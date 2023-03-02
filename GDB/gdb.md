# GDB

## 基本调试命令
运行gdb调试器来调试可执行程序
> gdb /path/program
> 
进入gdb调试命令行环境
> (gdb)
> 
基本命令

>r run 运行程序

>b break 打断点

>c continue 继续运行到下一个断点

>n next 单步执行,遇到子函数时不会进入子函数,而是将子函数整个执行完再停止

>s step 单步执行，遇到子函数就进入并且继续单步执行

>finish 继续运行，直到所选堆栈帧中的函数返回


## 多线程调试

### 常用线程调试命令
>info threads                  查看当前进程的线程
>thread \<ID\>                 切换调试的线程为指定ID的线程
>break test.c:100 thread all   在所有线程中相应的行上设置断点
>
>set scheduler-locking off|on
> off                         默认值，不锁定任何线程，所有线程都执行
> on                          只有当前被调试程序会执行

## segmentation-fault调试
抓取core文件
> ulimit -c unlimited

## gdbserver远程调试
在目标机上启动gdbserver
>gdbserver ip:port /path/program

在宿主机上启动gdb
>arm-linux-gdb /path/program

连接目标机
>target remote ip:port