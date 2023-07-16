# 高级I/O

## 非阻塞I/O

非阻塞I/O使我们可以发出open、read和write这样的I/O操作，并使这些操作不会永远阻塞。如果这种操作不能完成，则调用立即出错返回，表示该操作如继续执行将阻塞

对于一个给定的描述符，有两种为其指定非阻塞I/O的方法:

1. 如果调用open获得描述符，则可指定O_NONBLOCK标志

2. 对于已经打开的一个描述符，则可调用fcntl，由该函数打开 O_NONBLOCK 文件状态标志

有时，可以将应用程序设计成使用多线程的（见第11章），从而避免使用非阻塞I/O。如若我们能在其他线程中继续进行，则可以允许单个线程在I/O调用中阻塞

## 记录锁

当第一个进程正在读或修改文件的某个部分时，使用记录锁可以阻止其他进程修改同一文件区

### fcntl记录锁

```c
#include <fcnt1.h>

int fcnt1(int fd, int cmd, .../* struct flock *flockptr */);

//返回值：若成功，依赖于cmd（见下），否则，返回−1
```

cmd是F_GETLK、F_SETLK或F_SETLKW：

- F_GETLK 判断由flockptr所描述的锁是否会被另外一把锁所排斥（阻塞）。如果存在一把锁，它阻止创建由flockptr所描述的锁，则该现有锁的信息将重写flockptr指向的信息。如果不存在这种情况，则除了将l_type设置为F_UNLCK之外， flockptr所指向结构中的其他信息保持不变
- F_SETLK 设置由 flockptr 所描述的锁。如果我们试图获得一把读锁（l_type 为F_RDLCK）或写锁（l_type为F_WRLCK），而兼容性规则阻止系统给我们这把锁，那么fcntl会立即出错返回，此时errno设置为EACCES或EAGAIN。
- F_SETLKW 这个命令是F_SETLK的阻塞版本（命令名中的W表示等待（wait））。如果所请求的读锁或写锁因另一个进程当前已经对所请求区域的某部分进行了加锁而不能被授予，那么调用进程会被置为休眠。如果请求创建的锁已经可用，或者休眠由信号中断，则该进程被唤醒。

第三个参数（我们将调用flockptr）是一个指向flock结构的指针：
  
```c
struct flock {
    short l_type;　　　 /* F_RDLCK, F_WRLCK, or F_UNLCK */
    short l_whence;　　 /* SEEK_SET, SEEK_CUR, or SEEK_END */
    off_t l_start;　　　 /* offset in bytes, relative to l_whence */
    off_t l_len;　　　　 /* length, in bytes; 0 means lock to EOF */
    pid_t l_pid;　　　　 /* returned with F_GETLK */
};
```

记录锁的兼容性规则：

- 任意多个进程在一个给定的字节上可以有一把共享的读锁，但是在一个给定字节上只能有一个进程有一把独占写锁。
- 如果在一个给定字节上已经有一把或多把读锁，则不能在该字节上再加写锁；
- 如果在一个字节上已经有一把独占性写锁，则不能再对它加任何读锁。
- 如果一个进程对一个文件区间已经有了一把锁，后来该进程又企图在同一文件区间再加一把锁，那么新锁将替换已有锁

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
void lock_set(int fd, int type) 
{
    struct flock lock;
    lock.l_whence = SEEK_SET;
    lock.l_start = 0;
    lock.l_len = 0;
    while (1) {
        lock.l_type = type;
        if ((fcntl(fd, F_SETLK, &lock)) == 0) 
        {
            if (lock.l_type == F_RDLCK)
                printf("read lock set by %d\n", getpid());
            else if(lock.l_type == F_WRLCK)
                printf("write lock set by %d\n", getpid());
            else if (lock.l_type == F_UNLCK)
                printf("release lock by %d\n", getpid());
            return;
        }
        //检查文件是否可以上锁
        fcntl(fd, F_GETLK, &lock);
        //判断不能上锁的原因
        if (lock.l_type != F_UNLCK)
        {
            if (lock.l_type == F_RDLCK)
                printf("read lock has been already set by %d\n", lock.l_pid);
            else if (lock.l_type == F_WRLCK)
                printf("write lock has been already set by %d\n", lock.l_pid);
            getchar();
        }
    }
}
```

gcc lock_test.c -o write_lock

```c
int main()
{
    int fd;
    fd = open("data", O_RDWR | O_CREAT, 0666);
    if (fd < 0) {
        perror("open failed");
        return -1;
    }
    lock_set(fd, F_WRLCK); 
    getchar();
    lock_set(fd, F_UNLCK);
    getchar();
    close(fd);
    return 0;
}
```

gcc lock_test.c -o read_lock

```c
int main()
{
    int fd;
    fd = open("data", O_RDWR | O_CREAT, 0666);
    if (fd < 0) {
        perror("open failed");
        return -1;
    }
    
    lock_set(fd, F_RDLCK); 
    getchar();
    lock_set(fd, F_UNLCK);
    getchar();
    close(fd);
    return 0;
}
```

## I/O多路转接

多路I/O读写，

- 多进程。进程何时终止
- 多线程。线程间如何同步
- 轮询。等待浪费CPU时间
- 异步I/O。移植性成为一个问题；无法判别是哪一个描述符准备好了
- I/O多路转接：
    - 构造一张我们感兴趣的描述符（通常都不止一个）的列表
    - 调用一个函数，直到这些描述符中的一个已准备好进行I/O时，该函数才返回。进程会被告知哪些描述符已准备好可以进行I/O

### select和pselect

```c
#include <sys/select.h>

int select(int maxfdp1, fd_set *restrict readfds,
    fd_set *restrict writefds, fd_set *restrict exceptfds,
    struct timeval *restrict tvptr);
```
  

