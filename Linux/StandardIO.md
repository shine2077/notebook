# 标准 I/O

## 流`stream`和`FILE`对象

当我们使用标准I/O库打开或创建文件时，我们已经将一个流与一个文件关联起来。

## 打开关闭流

```c
#include <stdio.h>

FILE *fopen(const char *pathname, const char *mode);
```

```c
#include <stdio.h>

int fclose(FILE *stream);
```

## 读和写流

### 一次一个字符

#### 输入

```c
#include <stdio.h>

int fgetc(FILE *stream);

int getc(FILE *stream);

int getchar(void);
```
`getchar()`等同于`getc(stdin)`。而`getc`和`fgetc`的不同之处是前者可实现为宏，而后者必须为函数。

#### 输出

```c
#include <stdio.h>

int fputc(int c, FILE *stream);

int putc(int c, FILE *stream);

int putchar(int c);
```

同理`putchar(c)`等同于`putc(c, stdout)`。而`putc`和`fputc`的不同之处是前者实现为宏，而后者为函数。

### 一次一行字符

#### 输入

```c
#include <stdio.h>

char *fgets(char *s, int size, FILE *stream);

char *gets(char *s);

```

`gets`不推荐使用，因为该函数不指定输入缓冲区的字符长度，过长的字符被写入缓冲区后面的地址导致缓冲区溢出。

#### 输出

```c
#include <stdio.h>

int fputs(const char *s, FILE *stream);

int puts(const char *s);

```

## 二进制I/O

```c
#include <stdio.h>

size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);

size_t fwrite(const void *ptr, size_t size, size_t nmemb,
              FILE *stream);
```

如果成功，则`fread()`和`fwrite()`返回已读或已写的项数。只有当`size`为1时，这个数字才等于传输的字节数。

## 格式化IO

### 格式化输出

```c
#include <stdio.h>

int printf(const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
int dprintf(int fd, const char *format, ...);
int sprintf(char *str, const char *format, ...);
int snprintf(char *str, size_t size, const char *format, ...);

```


### 格式化输入

```c
#include <stdio.h>

int scanf(const char *format, ...);
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *str, const char *format, ...);

```

## 定位流

```c
#include <stdio.h>

int fseek(FILE *stream, long offset, int whence);
long ftell(FILE *stream);
void rewind(FILE *stream);

int fseeko(FILE *stream, off_t offset, int whence);
off_t ftello(FILE *stream);
```

- `ftell`和`fseek`假设文件的位置可以存储在一个长整数中,如果文件长度不适合用`long`类型来保存则考虑`fseeko`和`ftello`;
- 在某些体系结构上，`off_t`和`long`都是32位类型，如果`_FILE_OFFSET_BITS`(在包含任何头文件之前)定义为64 ，将把`off_t`转换为64位类型;


`rewind()`函数将流指向的文件位置设置为文件的开头。它相当于:

```c
(void) fseek(stream, 0L, SEEK_SET)
```

## 缓冲

合并系统调用

- 行缓冲：换行时刷新，满行时强制刷新，标准输出
- 全缓冲：满时强制刷新
- 无缓冲：立即刷新


强制刷新
```c
#include <stdio.h>

int fflush(FILE *stream);
```











