# 标准 I/O

## 流`stream`和文件对象`FILE`

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





