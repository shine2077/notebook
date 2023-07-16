
# XSI IPC

客户进程和服务器进程认同一个路径名和项目ID（项目ID是0～255之间的字符值），接着，调用函数ftok将这两个值变换为一个键。

```c
#include <sys/ipc.h>

key_t ftok(const char *path, int id);

//返回值：若成功，返回键；若出错，返回(key_t)−1
```

XSI IPC为每一个IPC结构关联了一个ipc_perm结构。该结构规定了权限和所有者，它至少包括下列成员：

```c
struct ipc_perm {
    uid_t uid; /* owner's effective user id */
    gid_t gid; /* owner's effective group id */
    uid_t cuid; /* creator's effective user id */
    gid_t cgid; /* creator's effective group id */
    mode_t mode; /* access modes */
};
```

## 消息队列

msqid_ds定义了队列的当前状态

```c
struct msqid_ds {
    struct ipc_perm　　 msg_perm;　　　　 /* see Section 15.6.2 */
    msgqnum_t　　　　　　msg_qnum;　　　　 /* # of messages on queue */
    msglen_t　　　　　　 msg_qbytes;　　　 /* max # of bytes on queue */
    pid_t　　　　　　　　msg_lspid;　　　　/* pid of last msgsnd() */
    pid_t　　　　　　　　msg_lrpid;　　　　/* pid of last msgrcv() */
    time_t　　　　　　　 msg_stime;　　　　/* last-msgsnd() time */
    time_t　　　　　　　 msg_rtime;　　　　/* last-msgrcv() time */
    time_t　　　　　　　 msg_ctime;　　　　/* last-change time */
};
```

打开一个现有队列或创建一个新队列

```c
#include <sys/msg.h>

int msgget(key_t key, int flag);

//返回值：若成功，返回消息队列ID；若出错，返回−1
```

`msgctl`函数对队列执行多种操作。

```c
#include <sys/msg.h>

int msgctl(int msqid, int cmd, struct msqid_ds *buf);

//返回值：若成功，返回0；若出错，返回−1 
```

cmd参数指定对`msqid`指定的队列要执行的命令:

- IPC_STAT 取此队列的`msqid_ds`结构，并将它存放在buf指向的结构中
- IPC_SET 将字段 `msg_perm.uid`、`msg_perm.gid`、`msg_perm.mode` 和 `msg_qbytes`从buf指向的结构复制到与这个队列相关的msqid_ds结构中。
- IPC_RMID 从系统中删除该消息队列以及仍在该队列中的所有数据。这种删除立即生效。

调用`msgsnd`将数据放到消息队列中

```c
#include <sys/msg.h>

int msgsnd(int msqid, const void *ptr, size_t nbytes, int flag);

//返回值：若成功，返回0；若出错，返回−1
```

ptr参数指向一个长整型数，它包含了正的整型消息类型`type`，其后紧接着的是消息数据（若nbytes是0，则无消息数据）

msgrcv从队列中取用消息。

```c
#include <sys/msg.h>
ssize_t msgrcv(int msqid, void *ptr, size_t nbytes, long type, int flag);
//返回值：若成功，返回消息数据部分的长度；若出错，返回-1
```

参数`type`可以指定想要哪一种消息:

- type == 0 返回队列中的第一个消息。
- type > 0 返回队列中消息类型为`type`的第一个消息。
- type < 0 返回队列中消息类型值小于等于 `type` 绝对值的消息，如果这种消息有若干个，则取类型值最小的消息。

### 消息队列与全双工管道的时间比较

在新的应用程序中不应当再使用它们

## 信号量

信号量并非是单个非负值，而必需定义为含有一个或多个信号量值的集合。当创建信号量时，要指定集合中信号量值的数量。

信号量的创建（semget）是独立于它的初始化（semctl）的。这是一个致命的缺点，因为不能原子地创建一个信号量集合，并且对该集合中的各个信号量值赋初值。

内核为每个信号量集合维护着一个semid_ds结构：

```c
 struct semid_ds {
    struct ipc_perm sem_perm;  /* Ownership and permissions */
    time_t          sem_otime; /* Last semop time */
    time_t          sem_ctime; /* Creation time/time of last
                                             modification via semctl() */
    unsigned long   sem_nsems; /* No. of semaphores in set */
    };
```

每个信号量由一个无名结构表示，它至少包含下列成员:

```c
struct {
    unsigned short　semval;　　　/* semaphore value, always >= 0 */
    pid_t　　　　　 sempid;　　　/* pid for last operation */
    unsigned short　semncnt;　　 /* # processes awaiting semval>curval */
    unsigned short　semzcnt;　　 /* # processes awaiting semval==0 */
};
```

### 函数semget来获得一个信号量ID

```c
#include <sys/sem.h>

int semget(key_t key, int nsems, int flag);

//返回值：若成功，返回信号量ID；若出错，返回−1
```

### semctl函数包含了多种信号量操作

```c
#include <sys/sem.h>

int semctl(int semid, int semnum, int cmd, ... /* union semun arg */);
```

第4个参数是可选的，是否使用取决于所请求的命令，如果使用该参数，则其类型是semun，它是多个命令特定参数的联合（union）：

```c
union semun {
    int　　　　　　　val;　　/* for SETVAL */
    struct semid_ds *buf; /* for IPC_STAT and IPC_SET */
    unsigned short *array; /* for GETALL and SETALL */
};
```

cmd参数指定下列10种命令:

- IPC_STAT 对此集合取semid_ds结构，并存储在由arg.buf指向的结构中。
- IPC_SET 按arg.buf指向的结构中的值，设置与此集合相关的结构中的sem_perm.uid、sem_perm.gid和sem_perm.mode字段。
- IPC_RMID 从系统中删除该信号量集合。这种删除是立即发生的。删除时仍在使用此信号量集合的其他进程，在它们下次试图对此信号量集合进行操作时，将出错返回EIDRM。
- GETVAL 返回成员semnum的semval值。
- SETVAL 设置成员semnum的semval值。该值由arg.val指定。
- GETPID 返回成员semnum的sempid值。
- GETNCNT 返回成员semnum的semncnt值。
- GETZCNT 返回成员semnum的semzcnt值。
- GETALL 取该集合中所有的信号量值。这些值存储在arg.array指向的数组中。
- SETALL 将该集合中所有的信号量值设置成arg.array指向的数组中的值。
对于除GETALL以外的所有GET命令，semctl函数都返回相应值。对于其他命令，若成功则返回值为0，若出错，则设置errno并返回−1。

### 函数semop对信号量集合由 semid 表示的信号量执行操作

```c
#include <sys/sem.h>
int semop(int semid, struct sembuf semoparray[], size_t nops);
//返回值：若成功，返回0；若出错，返回−1
```

参数semoparray是一个指针，它指向一个由sembuf结构表示的信号量操作数组：

```c
struct sembuf {
    unsigned short　　　sem_num;　　 /* member # in set (0, 1, ..., nsems-1 */
    short　　　　　　　　sem_op;　　　 /* operation(negative, 0,or pasitive )*/
    short　　　　　　　　sem_flg;　　 /* IPC_NOWAIT, SEM_UNDO */
};
```

#### 参数sem_op

sem_op 为正值。这对应于进程释放的占用的资源数。
sem_op 值会加到信号量的值上。如果指定了sem_flg成员为SEM_UNDO，则从该进程的此信号量中减去sem_op。

若sem_op为负值，则表示要获取由该信号量控制的资源。

- 如若该信号量的值大于等于 sem_op 的绝对值（具有所需的资源），则从信号量值中减去 sem_op的绝对值。这能保证信号量的结果值大于等于0。如果指定了sem_flg成员为SEM_UNDO，则从该进程的此信号量中加上sem_op。

- 如果信号量值小于sem_op的绝对值（资源不能满足要求），则适用下列条件：
a．若指定了IPC_NOWAIT，则semop出错返回EAGAIN。
b．若未指定IPC_NOWAIT，则该信号量的semncnt值加1（因为调用进程将进入休眠状态），然后调用进程被挂起直至下列事件之一发生。
i．此信号量值变成大于等于sem_op的绝对值（即某个进程已释放了某些资源）。此信号量的semncnt值减1（因为已结束等待），并且从信号量值中减去sem_op的绝对值。如果指定了undo标志，则sem_op的绝对值也加到该进程的此信号量调整值上。
ii．从系统中删除了此信号量。在这种情况下，函数出错返回EIDRM。
iii．进程捕捉到一个信号，并从信号处理程序返回，在这种情况下，此信号量的semncnt值减1（因为调用进程不再等待），并且函数出错返回EINTR。

- 若sem_op为0，这表示调用进程希望等待到该信号量值变成0。

## 共享存储

共享存储允许两个或多个进程共享一个给定的存储区。因为数据不需要在客户进程和服务器进程之间复制，所以这是最快的一种 IPC。

内核为每个共享存储段维护着一个结构：

```c
 struct shmid_ds {
    struct ipc_perm shm_perm;    /* Ownership and permissions */
    size_t          shm_segsz;   /* Size of segment (bytes) */
    time_t          shm_atime;   /* Last attach time */
    time_t          shm_dtime;   /* Last detach time */
    time_t          shm_ctime;   /* Creation time/time of last
                                    modification via shmctl() */
    pid_t           shm_cpid;    /* PID of creator */
    pid_t           shm_lpid;    /* PID of last shmat(2)/shmdt(2) */
    shmatt_t        shm_nattch;  /* No. of current attaches */
    ...
};
```

调用的第一个函数通常是`shmget`，它获得一个共享存储标识符。

`shmctl`函数对共享存储段执行多种操作

一旦创建了一个共享存储段，进程就可调用`shmat`将其连接到它的地址空间中。

当对共享存储段的操作已经结束时，则调用 `shmdt` 与该段分离。

# POXIX信号量

POSIX信号量接口意在解决XSI信号量接口的几个缺陷：

- POSIX 信号量接口使用更简单：没有信号量集。
- POSIX信号量在删除时表现更完美。回忆一下，当一个XSI信号量被删除时，使用这个信号量标识符的操作会失败，并将errno设置成EIDRM。使用POSIX信号量时，操作能继续正常工作直到该信号量的最后一次引用被释放

未命名信号量只存在于内存中，并要求能使用信号量的进程必须可以访问内存。这意味着它们只能应用在同一进程中的线程，或者不同进程中已经映射相同内存内容到它们的地址空间中的线程。

命名信号量可以通过名字访问，因此可以被任何已知它们名字的进程中的线程使用

initialize and open a named semaphore

```c
#include <fcntl.h>           /* For O_* constants */
#include <sys/stat.h>        /* For mode constants */
#include <semaphore.h>

sem_t *sem_open(const char *name, int oflag);
sem_t *sem_open(const char *name, int oflag,
                       mode_t mode, unsigned int value);
```

当使用一个现有的命名信号量时，我们仅仅指定两个参数：信号量的名字和 oflag 参数的 0值

当我们指定O_CREAT标志时，需要提供两个额外的参数:

- mode参数指定谁可以访问信号量
- value参数用来指定信号量的初始值

当完成信号量操作时，可以调用sem_close函数来释放任何信号量相关的资源。

```c
#include <semaphore.h>
int sem_close(sem_t *sem);
```

如果进程没有首先调用sem_close而退出，那么内核将自动关闭任何打开的信号量。注意，这不会影响信号量值的状态—如果已经对它进行了增1操作，这并不会仅因为退出而改变。

可以使用sem_unlink函数来销毁一个命名信号量。

```c
#include <semaphore.h>
int sem_unlink(const char *name);
```

sem_unlink函数删除信号量的名字。如果没有打开的信号量引用，则该信号量会被销毁。否则，销毁将延迟到最后一个打开的引用关闭。

可以使用sem_wait或者sem_trywait函数来实现信号量的减1操作。

```c
#include <semaphore.h>

int sem_trywait(sem_t *sem);\
int sem_wait(sem_t *sem);
```

使用`sem_wait`函数时，如果信号量计数是0就会发生阻塞。直到成功使信号量减1或者被信号中断时才返回。
调用`sem_trywait`时，如果信号量是0，则不会阻塞，而是会返回−1并且将errno置为EAGAIN

```c
#include <semaphore.h>
#include <time.h>

int sem_timedwait(sem_t *restrict sem, const struct timespec *restrict tsptr);
```

如果信号量可以立即减1，那么超时值就不重要了，尽管指定的可能是过去的某个时间，信号量的减 1 操作依然会成功。如果超时到期并且信号量计数没能减 1， sem_timedwait将返回-1且将errno设置为ETIMEDOUT

可以调用sem_post函数使信号量值增1。这和解锁一个二进制信号量或者释放一个计数信号量相关的资源的过程是类似的

```c
#include <semaphore.h>

int sem_post(sem_t *sem);
```

当我们想在单个进程中使用POSIX信号量时，使用未命名信号量更容易。可以调用sem_init函数来创建一个未命名的信号量。

```c
#include <semaphore.h>

int sem_init(sem_t *sem, int pshared, unsigned int value);
```

对未命名信号量的使用已经完成时，可以调用sem_destroy函数丢弃它。

```c
#include <semaphore.h>

int sem_destroy(sem_t *sem);
```
