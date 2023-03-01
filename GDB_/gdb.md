[toc]

# GDB基本调试命令


# 多线程调试
## 常用线程调试命令
>info threads                  查看当前进程的线程
>thread \<ID\>                 切换调试的线程为指定ID的线程
>break test.c:100 thread all   在所有线程中相应的行上设置断点
>
>set scheduler-locking off|on
> off                         默认值，不锁定任何线程，所有线程都执行
> on                          只有当前被调试程序会执行

# segmentation-fault调试
抓取core文件
> ulimit -c unlimited

# gdbserver远程调试

在目标机上启动gdbserver
>gdbserver ip:port /path/program

在宿主机上启动gdb
>arm-linux-gdb /path/program

连接目标机
>target remote ip:port