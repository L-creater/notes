# APUE

man 7 socket

man 7 epoll

man 7 tcp

读 ---name---SYNOPSIS   ---return value---error---conforming to---notes---see also---example

## I/O: input & output  ，

----一切实现的基础





​	stdio标准I/O

​    sysio系统调用IO   (都能实现时，优先标准IO)

### stdio:（5章） FILE类型贯穿始终

fopen(); -----man 3  fopen



> vim usr/include/asm-generic/erron
>
> man perror 
>
> man strerror
>
> ulimit -a
>
> 





fclose();



fgetc();

> man getchar
>
> man putchar

fputc();

fgets();

> man  gets

fputs();

fread();

> man fread
>
> man fwrite 

fwrite();



printf();

内存泄漏

scanf();



**用来操作文件位置指针**



fseek();

> 丑    long
>
> man  fseek
>
> 有创建---有删除
>
>  

ftell();

rewind();





fflush();

> 缓冲区作用：大多数情况下是好事，合并系统调用
>
> 行缓冲：换行时候刷新，满了的时候刷新，强制刷新（标准输出是这样的，----因为其是终端设备）
>
> 全缓冲：满了的时候刷新，强制刷新（默认，只要不是终端设备）
>
> 无缓冲：如 stderr ，需要立即输出的内容
>
> ​	setvbuf

getline 

临时文件：1、如何不冲突 	2、及时销毁

tmpnam

tmpfile

### sysio---文件IO/系统调用IO

fd是在文件IO中贯穿始终的类型



文件描述符的概念(整型数，数组下标，文件描述符优先使用当前可用范围内最小的)

FILE---（pos,fd）

文件IO操作：open,close,read,write,lseek

> man 2 open
>
> int open(const char *pathname, int flags);
>
> flags---位图--当前的使用权限
>
> 必须包含 R  W    RW	之一
>
> O_RDONLY  	O_WRONLY|O_CREAT|O_TRUNC	O_RDWR
>
> int open(const char *pathname, int flags，mode_t mode);
>
> 
>
> 有create  时用三参 	无时用二参
>
> c用的变参函数来实现---(不是函数重载)
>
> 如何验证----调open  传多个参数 4  5  6    报错 语法错误  则为定参函数----则是重载实现，没有报错则是---变参函数实现
>
> man  2	close
>
> man 	2	read
>
> man 2 write
>
> 
>
> ssize_t read(int fd, void *buf, size_t count);
>
> int fd---从这里读这个文件描述符
>
> void *buf----读到buf去
>
> size_t count----读count个
>
> /return value
>
> 
>
> ssize_t write(int fd, const void *buf, size_t count);
>
> int fd ----写一个文件描述符
>
> const void *buf-----消息来源
>
> size_t count-----写的大小
>
> /return 
>
> man 2 lseek
>
> 



文件IO与标准IO的区别

IO的效率问题

文件共享

原子操作

程序中的重定向：dup,dup2

同步：sync,fsync,fdatasync

fcntl();

ioctl();

/dev/fd/目录；





