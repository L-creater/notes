# 1 网络基础

## 分层模型结构：

​			OSI七层模型：物、数、网、传、会、表、应

​			TCP/IP四层模型：网（链路层/网络接口层）、网、传、应

​							应用层：http、ftp、nfs、ssh、telnet...

​							传输层：tcp、udp

​							 网络层：IP、ICMP、IGMP

​							 链路层：以太网帧协议、ARP

c/s模型：																			

​		client-server

b/s模型：

​		browser-server	

​								c/s																							b/s

​				优点：缓冲大量数据、协议选择灵活速度快									安全性、跨平台、开发工作量较小

​							

​				缺点：安全性																							不能缓存大量数据、严格遵守HTTP

​			

网络传输流程：

​			 数据没有封装之前，是不能在网络中传递。

​			 数据--应用层---传输层---网络层---链路层

以太网帧协议：

​				ARP协议，根据地址获取mac地址。

​			 	以太网帧协议，根据mac地址，完成数据包传输。

IP协议：

​			版本：IPv4、IPv6			---4位

​			TTL：time to live。 设置数据包在路由节点的跳转上限，没经过一个路由节点，该值-1，减为零的路由，有义务将该数据包丢弃。

​			源IP：32位。---4字节  	192.168.1.105---点分十进制 IP地址（实际是string）     ---二进制

IP地址：可以在网络环境中，唯一标识一台主机。

端口号：可以网络的一台主机上，唯一标识一个进程

IP地址+端口号：可以在网络环境中，唯一标识一个进程。

# 2 预备知识

UDP:

​		16位：源端口号			65536

​		16位：目的端口号		

IP协议：

​		16位：源端口号			65536

​		16位：目的端口号		

​		32序号			32确认序号	6个标志位	16位窗口大小。   65536

网络套接字：socket

​			一个文件描述符指向一个套接字（该套接字内部由内核借助两个缓冲区实现）。

​		在通信过程中，套接字一定是成对出现的。

![image-20220919150602032](E:\桌面\任务\unp\image-20220919150602032.png)

## 网络字节序：

​		小端法：（pc本地存储） 高位存高地址。地位存低地址。

​		大端法：（网络存储） 高位存低地址。地位存低地址。

> ​		atoi---string转int

> ​		h表示host，n表示network，l表示32位长整数，s表示16位短整数

​		htonl--->本地--》网络（IP）				192.168.230.198-->string-->atoi--->int--->htonl--->网络字节序

​		htons--->本地--》网络（port）

​		ntohl--->网络---》本地（IP）

​		ntohs--->网络---》本地（port）

## IP地址转换函数

```c
#include <arpa/inet.h>
	int inet_pton(int af, const char *src, void *dst);
	//af:AF_INET、AF_INET6
	//src:传入，IP地址（点分十进制）
	//dst:传出，转换后的 网络字节序的 IP地址
返回值：
    成功：1
    异常：0，说明src指向的不是一个有效的IP地址
    失败：-1
   
    
    
    const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
	//af:AF_INET、AF_INET6
	//src:传入，网络字节序IP地址
	//dst：本地字节序（string IP）
	//size：dst的大小
返回值：
    成功：dst
    失败：NULL
```

> p--ip  n--net

## sockaddr地址结构

```c
struct sockaddr_in addr;
addr.sin_family = AF_INET/AF_INET6;
addr.sin_port = htons(9527);		//端口号


int dst;
inet_pton(AF_INET,"192.168.230.198",(void*)dst);
addr.sin_addr.s_addr = dst;


[***]addr.sin_addr.s_addr = htonl(INADDR_ANY);  //取出系统中任意IP地址，二进制类型
bind(fd,(struct sockaddr *)&addr,size);
//***  man 7 ip

```

# 网络套接字函数

## socket函数

```c
int toupper(int c);//--小写变大写


#include <sys/socket.h>
int socket(int domain, int type, int protocol);
    //domain: AF_INET、AF_INET6、AF_UNIX
	//type:数据传输协议  SOCK_STREAM 这个协议是按照顺序的、可靠的、数据完整的基于字节流的连接。这是一个使			用最多的socket类型，这个socket是使用TCP来进行传输。
	//SOCK_DGRAM 这个协议是无连接的、固定长度的传输调用。该协议是不可靠的，使用UDP来进行它的连接。
	//protocol: 默认  0
返回值：
	成功：返回指向新创建的socket的文件描述符，失败：返回-1，设置errno
    
```

## bind函数

> 给socket绑定（  IP和  port）----地址结构

```c
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
sockfd：
	socket 函数返回值 
    
    struct sockaddr_in addr;
	addr.sin_family = AF_INET;
	addr.sin_port = htons(666); //绑定端口号
	addr.sin_addr.s_addr = htonl(INADDR_ANY);
addr: (struct sockaddr *)&addr
	构造出IP地址加端口号
addrlen:
	sizeof(addr)长度---地址结构大小
返回值：
	成功返回0，失败返回-1, 设置errno

```



## listen函数

> 设置同时与服务器建立连接的上限数，（同时进行3次握手的客户端数量）

```c
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int listen(int sockfd, int backlog);
sockfd:
	socket 函数返回值
backlog:
	上限数值
	最大值 128；
返回值：
    成功返回0，失败返回-1。
```



## accept函数

> 阻塞等待客户端建立链接，成功的话，返回一个与客户端成功连接的socket文件描述符。
>
> man 2 accpet

```c
#include <sys/types.h> 		/* See NOTES */
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
sockdf:
	socket 函数返回值
addr:
	传出参数。返回成功与服务器建立链接的那个客户端的地址结构（含IP地址和端口号）
        
   socklen_t clit_addr_len = sizeof(addr);
addrlen: &clit_addr_len
	传入传出参数（值-结果）,入sizeof(addr)大小，出：客户端addr实际大小。
    
返回值：
	能与服务器进行通讯的的socket文件描述符，失败返回-1，设置errno

```



## connect函数

> ​		使用现有的socket与服务器建立连接

```c
#include <sys/types.h> 					/* See NOTES */
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
sockdf:
	socket 函数返回值
addr:
	传入参数，指定服务器端地址结构，含IP地址和端口号
addrlen:
	服务器地址结构大小
	传入参数,传入sizeof(addr)大小
返回值：
	成功返回0，失败返回-1，设置errno

   //如果不使用bind绑定客户端地址结构，采用“隐式绑定”
```

> TCP通信流程分析
>
> ​		server：
>
> ​				1.socket()		  创建socket
>
> ​				2.bind()			 绑定服务器地址结构
>
> ​				3.listen()		    设置监听上限
>
> ​				4.accept()		  阻塞监听客户端连接
>
> ​				5.read(fd)		 读socket获取客户端数据
>
> ​				6.小写---大写		toupper
>
> ​				7.write(fd)					
>
> ​				8.close()
>
> ​		client:
>
> ​					1.socket()				 创建socket
>
> ​					2.connect()			 与服务器建立连接
>
> ​					3.write()				  转换后的数据
>
> ​					4.read()				  都转换后的数据
>
> ​					5.显示读取结果
>
> ​					6.close()

​	

# 出错处理封装函数

封装的目的：

​				在server.c编程过程中，将出错处理与逻辑分开，可以直接跳转man手册

【wrap.c】																												  【wrap.h】

存放网络通信相关常用  自定义函数																	存放网络通信相关常用   自定义函数原型（声明）																					

命名方式：系统调用函数首字符大写，方便查看man手册

函数功能：调用系统调用函数、出错处理场景

在server.c和client.c生成server

联合编译 server.c 和 wrap.c 生成server

​				 

## wrap.h

```c
#ifndef __WRAP_H_
#define __WRAP_H_
void perr_exit(const char *s);
int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr);
int Bind(int fd, const struct sockaddr *sa, socklen_t salen);
int Connect(int fd, const struct sockaddr *sa, socklen_t salen);
int Listen(int fd, int backlog);
int Socket(int domain, int type, int protocol);
ssize_t Read(int fd, void *ptr, size_t nbytes);
ssize_t Write(int fd, const void *ptr, size_t nbytes);
int Close(int fd);
ssize_t Readn(int fd, void *vptr, size_t n);
ssize_t Writen(int fd, const void *vptr, size_t n);
ssize_t my_read(int fd, char *ptr);
ssize_t Readline(int fd, void *vptr, size_t maxlen);
#endif

```



## wrap.c

```c

#include <stdlib.h>
#include <errno.h>
#include <sys/socket.h>
void perr_exit(const char *s)
{
	perror(s);
	exit(1);
}
int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr)
{
	int n;
	again:
	if ( (n = accept(fd, sa, salenptr)) < 0) {
		if ((errno == ECONNABORTED) || (errno == EINTR))
			goto again;
		else
			perr_exit("accept error");
	}
	return n;
}
int Bind(int fd, const struct sockaddr *sa, socklen_t salen)
{
	int n;
	if ((n = bind(fd, sa, salen)) < 0)
		perr_exit("bind error");
	return n;
}
int Connect(int fd, const struct sockaddr *sa, socklen_t salen)
{
	int n;
	if ((n = connect(fd, sa, salen)) < 0)
		perr_exit("connect error");
	return n;
}
int Listen(int fd, int backlog)
{
	int n;
	if ((n = listen(fd, backlog)) < 0)
		perr_exit("listen error");
	return n;
}
int Socket(int domain, int type, int protocol)
{
	int n;
	if ( (n = socket(domain, type, protocol)) < 0)
		perr_exit("socket error");
	return n;
}
ssize_t Read(int fd, void *ptr, size_t nbytes)
{
	ssize_t n;
again:
	if ( (n = read(fd, ptr, nbytes)) == -1) {
		if (errno == EINTR)
			goto again;
		else
			return -1;
	}
	return n;
}
ssize_t Write(int fd, const void *ptr, size_t nbytes)
{
	ssize_t n;
again:
	if ( (n = write(fd, ptr, nbytes)) == -1) {
		if (errno == EINTR)
			goto again;
		else
			return -1;
	}
	return n;
}
int Close(int fd)
{
	int n;
	if ((n = close(fd)) == -1)
		perr_exit("close error");
	return n;
}
//参3---应该读取的字节数
ssize_t Readn(int fd, void *vptr, size_t n)
{
	size_t nleft;
	ssize_t nread;
	char *ptr;

	ptr = vptr;
	nleft = n;

	while (nleft > 0) {
		if ( (nread = read(fd, ptr, nleft)) < 0) {
			if (errno == EINTR)
				nread = 0;
			else
				return -1;
		} else if (nread == 0)
			break;
		nleft -= nread;
		ptr += nread;
	}
	return n - nleft;
}
//参3---写出字节数
ssize_t Writen(int fd, const void *vptr, size_t n)
{
	size_t nleft;
	ssize_t nwritten;
	const char *ptr;

	ptr = vptr;
	nleft = n;

	while (nleft > 0) {
		if ( (nwritten = write(fd, ptr, nleft)) <= 0) {
			if (nwritten < 0 && errno == EINTR)
				nwritten = 0;
			else
				return -1;
		}
		nleft -= nwritten;
		ptr += nwritten;
	}
	return n;
}
//
static ssize_t my_read(int fd, char *ptr)
{
	static int read_cnt;
	static char *read_ptr;
	static char read_buf[100];

	if (read_cnt <= 0) {
again:
		if ((read_cnt = read(fd, read_buf, sizeof(read_buf))) < 0) {
			if (errno == EINTR)
				goto again;
			return -1;	
		} else if (read_cnt == 0)
			return 0;
		read_ptr = read_buf;
	}
	read_cnt--;
	*ptr = *read_ptr++;
	return 1;
}
//readline---fgetS
ssize_t Readline(int fd, void *vptr, size_t maxlen)
{
	ssize_t n, rc;
	char c, *ptr;
	ptr = vptr;

	for (n = 1; n < maxlen; n++) {
		if ( (rc = my_read(fd, &c)) == 1) {
			*ptr++ = c;
			if (c == '\n')
				break;
		} else if (rc == 0) {
			*ptr = 0;
			return n - 1;
		} else
			return -1;
	}
	*ptr = 0;
	return n;
}

```

# 高并发服务器

## 多进程并发服务器---server

```c
1.Socket();					创建  监听套接字 lfd
2.Bind();					绑定地址结构  Struct  sockaddr_in addr
3.Listen();				  

4.while(1)
	{
		cfd = Accept();				//接收客户端请求
    	pid = fork();
    	if(pid == 0)				//子进程read(cfd)--小--》大
        {
            close(lfd);				//关闭用于建立连接的套接字lfd
            read();
            小---大
            write();
        }
    	else if(pid > 0)
        {
            close(cfd);
        
            
            contiue;
        }
								
	 }
5.子进程：
   			close(lfd);				
            read();
            小---大
            write();
  父进程：
      close(cfd);
      注册信号捕获函数：  SIGCHLD
      在回调函数中，完成子进程回收
          while(waitpid());
       
```

## 多线程并发服务器

```c
1.Socket();					创建  监听套接字 lfd
2.Bind();					绑定地址结构  Struct  sockaddr_in addr
3.Listen();				  

4.while(1)
	{
		cfd = Accept();				//接收客户端请求
    	pthread_create(&tid,NULL,tfn,NULL);
		pthread_detach(tid);		//pthead_join(tid,void **);新线程---专门用于回收子进程				
	 }
5.子线程：
 	void *tfn(void *arg)
	{
    		close(lfd);				//关闭用于建立连接的套接字lfd
            read(cfd);
            小---大
            write(cfd);
    		pthread_exit((void *)10);
	}
);
       
```

readn----读n个字节

readline---读一行





read 函数返回值：

   1. >0 实际独到的字节数
      >
      >= 0 已经读到结尾(对端已经关闭)
      >
      >-1  应进一步判断error的值
      >
      >​			error = EAGAIN  OR EWOULDBLOCK:设置了非阻塞方式 读，没有数据到达
      >
      >​			error = EINTR 慢速系统调用被中断
      >
      >​			error = "其他情况"  异常

```c

#define  SRV_PORT 1024;

void catch_child(int signum)
{
    while(waitpid(0,NULL,WNOHANG) > 0)  //头文件
    return ;
    
}
int main()
{
    int lfd, cfd;
    pid_t pid;
    int ret,i;
    char buf[BUFSIZ];
    struct sockaddr_in srv_addr,clt_addr;
    socklen_t clt_addr_len;
   // meset(&sre_addr, 0, sizeof(srv_addr));
    bzero(&srv_addr,sizeof(srv_addr)); //置零
    srv_addr.sin_family=AF_INET;
    srv_addr.sin_port = htons(SRV_PORT);
    srv.addr.sin_addr.s_addr = htonl(INADDR_ANY);
    
    lfd = socket(AF_INET,SOCK_STREAM,0);
    Bind(lfd,(struct sockaddr *)&srv_addr,sizeof(srv_addr));
    Listen(lfd,128);
    
    clt_addr_len = sizeof(clt_addr);
    while(1)
    {
        
       cfd = Accept(lfd,(struct sockaddr *)&clt_addr,&clt_addr_len);
        //创建子进程
        pid = fork();
        if(pid < 0);
        perr_exit("fock error");
        else if(pid == 0)
        {
            close(lfd);
            break;
        }
        else
        {
            struct sigaction act;
            act.sa_handler = catch_child;
            sigemptyset(&act.sa_mask);
            act.sa_flags = 0;
            
            sigaction(SIGCHLD,&act,NULL);
            if(ret != 0)
            {
                perr_exit("sigaction error");
            }
            close(cfd);
            contiue;
        }
    }
    if(pid == 0)
    {
        for(;;)
        {
            ret = read(vfd,buf,sizeof(buf));
            if(ret == 0)
            {
                close(cfd);
                exit(1);
            }
            for(i = 0; i < ret; i++)
            {
                buf[i] = toupper(buf[i]);

            }
            write(cfd,buf,ret);
            write(STDOUT_FILENO,buf,ret);

        }
    }
}


//scp  
```

端口复用：

​	

```c
int opt = 1;	//设置端口复用
setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, (void *)&opt,sizeof(opt));


//return---
```

> man  2  shutdown
>
> int shutdown(int sockfd, int how);
>
> ​							how:   SHUT_RD      		关读
>
> ​										SHUT_WR			 关写							
>
> ​										SHUT_RDWR		关读写

> close--调用一次， 文件描述符减一
>
> shutdown---文件描述符，---全部关闭

> 重点---TCP状态时序图--结合三次握手和四次挥手理解记忆
>
> netstat   -apn|grep 端口号
>
> 



## 多路I/O转接服务器

### select多路IO转接

​					原理：借助内核，select来监听，客户端连接

```c
#include <sys/select.h>
/* According to earlier standards */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

	nfds: 		监控的文件描述符集里最大文件描述符加1，因为此参数会告诉内核检测前多少个文件描述符的状态
	readfds：	监控有读数据,到达文件描述符集合，传入传出参数
	writefds：	监控写数据,到达文件描述符集合，传入传出参数
	exceptfds：	监控异常发生，到达文件描述符集合,如带外数据到达异常，传入传出参数
	timeout：	定时阻塞监控时间，3种情况
				1.NULL，永远等下去
				2.设置timeval，等待固定时间
				3.设置timeval里时间均为0，非阻塞监听，检查描述字后立即返回，轮询。
        
返回值：
        > 0 :所有监听集合(3个)中，满足对应事件的总数。
          0 :没有满足监听条件的文件描述符
          -1:errno
            
        
	struct timeval {
		long tv_sec; /* seconds */
		long tv_usec; /* microseconds */
	};
	void FD_CLR(int fd, fd_set *set); 	//把文件描述符集合里fd清0
	int FD_ISSET(int fd, fd_set *set); 	//测试文件描述符集合里fd是否置1
	void FD_SET(int fd, fd_set *set); 	//把文件描述符集合里fd位置1
	void FD_ZERO(fd_set *set); 			//把文件描述符集合里所有位清0

```

```c
void FD_ZERO(fd_set *set); 			//清空一个文件描述符集合
fd_set rset;
FD_ZERO(&rset);

void FD_SET(int fd, fd_set *set); 	//将待监听的文件描述符，添加到监听集合中
FD_SET(3,&rset);
FD_SET(4,&rset);
FD_SET(5,&rset);

void FD_CLR(int fd, fd_set *set); 	//将一个文件描述符从监听集合中，移除
FD_CLR(4,&rset);

int FD_ISSET(int fd, fd_set *set); 	//判断一个文件描述符是否在监听集合中
返回值：在--1，不在--0
    FD_ISSET(&rset);
	


```

思路分析：

```c
	
	lfd = socket();				//创建套接字
	bind();						//绑定地址结构
	fd_set rset,allset;				//设置监听上限	
	FD_ZERO(&allrset);	//将读监听集合清空
	FD_SET(lfd,&allrset);			//将lfd添加至读集合中

	while(1)
    {
        rset = allset;
        ret = select(lfd+1,&rset,NULL,NULL,NULL);//监听文件描述符对应的集合
        if(ret > 0)
        {
            if(FD_ISSET(lfd, &ret))
            {
                cfd = accept();
                FD_SET(cfd,&allrset);
            }
            for(i = lfd+1; i <=最大文件描述符；i++)
            {
                FD_ISSET(i, &rset);
                read();
                小---大；
                write();
                
                          
            }
        }
    }

```

```c
select
1.	select能监听的文件描述符个数受限于FD_SETSIZE,一般为1024，单纯改变进程打开的文件描述符个数并不能改变select监听文件个数
2.	解决1024以下客户端时使用select是很合适的，但如果链接客户端过多，select采用的是轮询模型，会大大降低服务器响应效率，不应在select上投入更多精力
#include <sys/select.h>
/* According to earlier standards */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int nfds, fd_set *readfds, fd_set *writefds,
			fd_set *exceptfds, struct timeval *timeout);

	nfds: 		监控的文件描述符集里最大文件描述符加1，因为此参数会告诉内核检测前多少个文件描述符的状态
	readfds：	监控有读数据到达文件描述符集合，传入传出参数
	writefds：	监控写数据到达文件描述符集合，传入传出参数
	exceptfds：	监控异常发生达文件描述符集合,如带外数据到达异常，传入传出参数
	timeout：	定时阻塞监控时间，3种情况
				1.NULL，永远等下去
				2.设置timeval，等待固定时间
				3.设置timeval里时间均为0，检查描述字后立即返回，轮询
	struct timeval {
		long tv_sec; /* seconds */
		long tv_usec; /* microseconds */
	};
	void FD_CLR(int fd, fd_set *set); 	//把文件描述符集合里fd清0
	int FD_ISSET(int fd, fd_set *set); 	//测试文件描述符集合里fd是否置1
	void FD_SET(int fd, fd_set *set); 	//把文件描述符集合里fd位置1
	void FD_ZERO(fd_set *set); 			//把文件描述符集合里所有位清0
server
/* server.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	int i, maxi, maxfd, listenfd, connfd, sockfd;
	int nready, client[FD_SETSIZE]; 	/* FD_SETSIZE 默认为 1024 */
	ssize_t n;
	fd_set rset, allset;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN]; 			/* #define INET_ADDRSTRLEN 16 */
	socklen_t cliaddr_len;
	struct sockaddr_in cliaddr, servaddr;

	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port = htons(SERV_PORT);

Bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

Listen(listenfd, 20); 		/* 默认最大128 */

maxfd = listenfd; 			/* 初始化 */
maxi = -1;					/* client[]的下标 */

for (i = 0; i < FD_SETSIZE; i++)
	client[i] = -1; 		/* 用-1初始化client[] */

FD_ZERO(&allset);
FD_SET(listenfd, &allset); /* 构造select监控文件描述符集 */

for ( ; ; ) {
	rset = allset; 			/* 每次循环时都从新设置select监控信号集 */
	nready = select(maxfd+1, &rset, NULL, NULL, NULL);

	if (nready < 0)
		perr_exit("select error");
	if (FD_ISSET(listenfd, &rset)) { /* new client connection */
		cliaddr_len = sizeof(cliaddr);
		connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
		printf("received from %s at PORT %d\n",
				inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
				ntohs(cliaddr.sin_port));
		for (i = 0; i < FD_SETSIZE; i++) {
			if (client[i] < 0) {
				client[i] = connfd; /* 保存accept返回的文件描述符到client[]里 */
				break;
			}
		}
		/* 达到select能监控的文件个数上限 1024 */
		if (i == FD_SETSIZE) {
			fputs("too many clients\n", stderr);
			exit(1);
		}

		FD_SET(connfd, &allset); 	/* 添加一个新的文件描述符到监控信号集里 */
		if (connfd > maxfd)
			maxfd = connfd; 		/* select第一个参数需要 */
		if (i > maxi)
			maxi = i; 				/* 更新client[]最大下标值 */

		if (--nready == 0)
			continue; 				/* 如果没有更多的就绪文件描述符继续回到上面select阻塞监听,
										负责处理未处理完的就绪文件描述符 */
		}
		for (i = 0; i <= maxi; i++) { 	/* 检测哪个clients 有数据就绪 */
			if ( (sockfd = client[i]) < 0)
				continue;
			if (FD_ISSET(sockfd, &rset)) {
				if ( (n = Read(sockfd, buf, MAXLINE)) == 0) {
					Close(sockfd);		/* 当client关闭链接时，服务器端也关闭对应链接 */
					FD_CLR(sockfd, &allset); /* 解除select监控此文件描述符 */
					client[i] = -1;
				} else {
					int j;
					for (j = 0; j < n; j++)
						buf[j] = toupper(buf[j]);
					Write(sockfd, buf, n);
				}
				if (--nready == 0)
					break;
			}
		}
	}
	close(listenfd);
	return 0;
}

```



> ​		select优缺点：
>
> ​					缺点：监听上线受文件描述符限制，最大1024
>
> ​								检测满足条件的fd,自己添加业务逻辑提高较少。提高了编码难度
>
> ​					优点：跨平台。win、linux、unix、macOS...

### poll



```c
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
	fds:监听的文件描述符【数组】
	struct pollfd {
		int fd; /* 文件描述符 */
		short events; /* 监控的事件 */
		short revents; /* 监控事件中满足条件返回的事件 返回非零---》POLLIN、POLLOUT、POLLERR*/
	};
	POLLIN			普通或带外优先数据可读,
	即POLLRDNORM | POLLRDBAND
	POLLRDNORM		数据可读
	POLLRDBAND		优先级带数据可读
	POLLPRI 		高优先级可读数据
	POLLOUT			普通或带外数据可写
	POLLWRNORM		数据可写
	POLLWRBAND		优先级带数据可写
	POLLERR 		发生错误
	POLLHUP 		发生挂起
	POLLNVAL 		描述字不是一个打开的文件

	nfds 			监控数组中有多少文件描述符需要被监控---实际有效监听个数。

	timeout 		毫秒级等待
		-1：阻塞等待，#define INFTIM -1 				Linux中没有定义此宏
		0：立即返回，不阻塞进程
		>0：等待指定毫秒数，如当前系统时间精度不够毫秒，向上取值
返回值：
        返回满足对应监听事件的文件描述符总个数。

```

### read

```c
****read函数返回值：
	 > 0:实际读到的字节数
     = 0： socket中，表示对端关闭。需要重启。
     -1 ： 如果: error == EINTR 被异常终端，需要重启
         	  	error == EAGIN 或 EWOULDBLOCK 以非阻塞方式读取数据，但是没有数据需要再次读。
        		error == ECONNRESET 说明连接被重置。需要close() ,移除监听队列
         
         都不是  ---返回错误
```

> 优点： 自带数据结构，可以将监听事件集合和返回事件集合 分离
>
> ​			拓展 监听上限，超出1024设置
>
> 缺点：不能跨平台。Linux   无法直接定位满足监听事件的文件描述符，编码难度较大。

### ***epoll

> 突破1024文件描述符限制：
>
> ​			cat	/proc/sys/fs/file-max  -->当前计算机所能打开的最大文件个数。受硬件影响
>
> 修改：
>
> ​			打开sudo vi /etc/security/limits.conf   写入
>
>    * soft  nofile 65536		---设置默认值，可以直接接助命令修改 
> 			* hard nofile 100000      ---命令修改上限

> 优点：高效，突破1024文件描述符
>
> 缺点：不能跨平台

​		epoll是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率，因为它会复用文件描述符集合来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。

目前epell是linux大规模并发网络程序中的热门首选模型。

epoll除了提供select/poll那种IO事件的电平触发（Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

```c
#include <sys/epoll.h>
//红黑树  平衡二叉树

int epoll_create(int size)		---创建一棵红黑树
    size：创建红黑树的监听数目（仅供内核参考）
	返回值：指向新创建红黑树的根节点 的fd
    	  失败： -1 errno
    
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)---控制红黑树
		epfd：	为epoll_creat的返回值
		op：		表示动作，用3个宏来表示：
                 EPOLL_CTL_ADD (添加fd到  监听epo红黑树)，
                 EPOLL_CTL_MOD (修改已经注册的fd在红黑树上的监听事件)，
                 EPOLL_CTL_DEL (从epfd除一个fd---将一个fd从监听的红黑树上摘下(取消监听))；
    	fd:	待监听的fd
            
		event：	本质  struct epoll_event  结构体地址
            	events:
						EPOLLIN  
                        EPOLLOUT
                        EPOLLERR
            	data: 联合体
                      int fd;
					  void *ptr;
					  uint_32 32;
					  uint_64 64;
		返回值：成功 0；失败-1  errno

		struct epoll_event {
			__uint32_t events; /* Epoll events */
			epoll_data_t data; /* User data variable */
		};
		typedef union epoll_data {
			void *ptr;
			int fd;
			uint32_t u32;
			uint64_t u64;
		} epoll_data_t;

		EPOLLIN ：	表示对应的文件描述符可以读（包括对端SOCKET正常关闭）
		EPOLLOUT：	表示对应的文件描述符可以写
		EPOLLPRI：	表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）
            
            
            
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)
		events：		传出参数，传出满足监听条件  fd的结构体---用来存内核得到事件的集合，可以简单看作数组[]。
		maxevents：	告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，
            struct epoll_event events[1024];
			maxevents--1024(最大)
            
		timeout：---	是超时时间
			-1：	阻塞
			0：	立即返回，非阻塞
			>0：	指定毫秒
		
返回值：	成功返回有多少文件描述符就绪，时间到时返回0，出错返回-1
			>0  ---可以用作循环上限
			0 ： 没有fd满足监听事件
            -1  
```

```c
//思路分析---epoll实现多路IO转接
lfd = socket();
bind();
listen();

int epfd = epoll_create(1024);	//监听红黑树的树根
struct epoll_event tep,ep[1024];	//设置单个fd属性，ep 是epoll_wait()传出的满足监听事件的数组

tep.events = EPOLLIN;	初始化  lfd的监听属性
tep.data.fd = lfd;

  epoll_ctl(epfd, EPOLL_CTL_ADD,cfd,&tep);
    
    

epoll_ctl(epfd,EPOLL_CTL_ADD,lfd); //将lfd添加到监听红黑树上
while(1)
{
    ret = epoll_wait(epfd, ep, 1024,-1);  //ret满足监听事件总个数
    for(i = 0; i< ret; ++i)
    {
        if(ep[i].data.fd == lfd)	//lfd满足读事件，有新的客户端发起连接请求
        {
            cfd = accept();
            epoll_ctl(epfd, EPOLL_CTL_ADD,cfd,&tep);
        }
        else
        {		//cfd 满足读事件，有客户端写数据来
            n=read(ep[i].data.fd,buf,sizeof(buf));
            if(n == 0)
            {
                close(ep[i].data.fd);
                epoll_ctl(epfd, EPOLL_CTL_DEL,ep[i].data.fd,NULL);
            }
            else if(n > 0)
            {
                小---大
                write(ep[i].data.fd,buf,n);
            }
        }
       
    }
}



    
```

#### epoll事件

##### ET模式

​			边沿触发:---缓冲区剩余未读尽的数据不会导致epoll_wait返回。新的事件满足，才会触发

​			

```c
struct epoll_event event;
event.events = EPOLLIN | EPOLLET; 
```

ET模式： 高效模式，但是只支持非阻塞模式---忙轮询

```c
struct epoll_event event;
event.events = EPOLLIN | EPOLLET; 
epoll_ctl(epfd,EPOLL_CTL_ADD,cfd,&event);
int flg = fcnl(cfd,F_GETFL);
flg |= O_NONBLOCK;
fcntl(cfd, F_SETFL, flg);
```



```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/epoll.h>
#include <errno.h>
#include <unistd.h>

#define MAXLINE 10
//进程间通信
int main(int argc, char *argv[])
{
	int efd, i;
	int pfd[2];
	pid_t pid;
	char buf[MAXLINE], ch = 'a';

	pipe(pfd);
	pid = fork();
	if (pid == 0)	//子进程  子写
    {
		close(pfd[0]);
		while (1) 
        {
            //aaa\n
			for (i = 0; i < MAXLINE/2; i++)
				buf[i] = ch;
			buf[i-1] = '\n';
			ch++;//a++ == b
			//bbbb\n
			for (; i < MAXLINE; i++)
				buf[i] = ch;
			buf[i-1] = '\n';
			ch++;
			//aaaa\nbbbb\n
			write(pfd[1], buf, sizeof(buf));
			sleep(5);
		}
		close(pfd[1]);
	} 
    else if (pid > 0) 	//父进程  父读
    {
		struct epoll_event event;
		struct epoll_event resevent[10];
		int res, len;
		close(pfd[1]);

		efd = epoll_create(10);
		/* event.events = EPOLLIN; *///LT水平触发(默认)
		event.events = EPOLLIN | EPOLLET;		/* ET 边沿触发 ，默认是水平触发 */
		event.data.fd = pfd[0];
		epoll_ctl(efd, EPOLL_CTL_ADD, pfd[0], &event);

		while (1) 
        {
			res = epoll_wait(efd, resevent, 10, -1);
			printf("res %d\n", res);
			if (resevent[0].data.fd == pfd[0])
            {
				len = read(pfd[0], buf, MAXLINE/2);
				write(STDOUT_FILENO, buf, len);
			}
		}
		close(pfd[0]);
		close(efd);
	} 
    else 
    {
		perror("fork");
		exit(-1);
	}
	return 0;
}

```



##### LT模式

​			水平触发---默认方式			

​							缓冲区剩余未读尽的数据会导致epoll_wait返回

##### 实例2：

###### server

```c
/* server.c */
#include <stdio.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <signal.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/epoll.h>
#include <unistd.h>

#define MAXLINE 10
#define SERV_PORT 8080

int main(void)
{
	struct sockaddr_in servaddr, cliaddr;
	socklen_t cliaddr_len;
	int listenfd, connfd;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int i, efd;

	listenfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	listen(listenfd, 20);

	struct epoll_event event;
	struct epoll_event resevent[10];
	int res, len;
	efd = epoll_create(10);
	event.events = EPOLLIN | EPOLLET;		/* ET 边沿触发 ，默认是水平触发 */

	printf("Accepting connections ...\n");
	cliaddr_len = sizeof(cliaddr);
	connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
	printf("received from %s at PORT %d\n",
			inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
			ntohs(cliaddr.sin_port));

	event.data.fd = connfd;
	epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &event);

	while (1) {
		res = epoll_wait(efd, resevent, 10, -1);
		printf("res %d\n", res);
		if (resevent[0].data.fd == connfd) {
			len = read(connfd, buf, MAXLINE/2);
			write(STDOUT_FILENO, buf, len);
		}
	}
	return 0;
}

```

###### client

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>

#define MAXLINE 10
#define SERV_PORT 8080

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, i;
	char ch = 'a';

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (1) {
		for (i = 0; i < MAXLINE/2; i++)
			buf[i] = ch;
		buf[i-1] = '\n';
		ch++;

		for (; i < MAXLINE; i++)
			buf[i] = ch;
		buf[i-1] = '\n';
		ch++;

		write(sockfd, buf, sizeof(buf));
		sleep(10);
	}
	Close(sockfd);
	return 0;
}

```

epoll 反应堆模型：

​			epoll	ET模式 + 非阻塞+轮询 + void *ptr

原来： socket、bind、listen、---epoll_creat 监听红红黑树 -- 返回 epfd --- epoll_ctl() 向树上添加一个监听fd ---while(1)---epoll_wait 监听 ---对应监听fd有事件发生 -- 返回 监听满足数组。 --判断返回数组元素 --lfd满足 --accept  ---fcd满足--read()  --小》大 ---write 回去



反应堆：

​			 socket、bind、listen、---epoll_creat 监听红红黑树 -- 返回 epfd --- epoll_ctl() 向树上添加一个监听fd ---while(1)---epoll_wait 监听 ---对应监听fd有事件发生 -- 返回 监听满足数组。 --判断返回数组元素 --lfd满足 --accept  ---fcd满足--read()  --小》大 ---cfd从监听红红黑树上摘下 ---epoll_ctl()监听cfd写事件---EPOLLOUT--回调函数--epoll_ctl()---EPOLL_CTL_ADD 重新放到红黑树上监听写事件--等待epoll_wait返回--说明cfd可写---write回去---cfd从监听红红黑树上摘下 --EPOLLIN--epoll_ctl()---EPOLL_CTL_ADD 重新放到红黑树上监听写事件--epoll_wait监听

---write 回去



eventset函数：
				设置回调函数。lfd--->acceptconn()

​											 cfd--->recvdata()

eventadd函数：	

​				将一个fd，添加到红黑树。设置监听read事件，还是监听写事件。

网络编程中：  		read ---recv()

​									writr---

> ctags 
>
> 生成方法：1.在项目目录下  ctags ./*  -R
>
> ​					2.在任意一个文件内使用   ctrl + p 创建tags
>
> 命令：
>
> ​		ctrl +]  ---光标放置于调用函数上，跳转至函数定义。
>
> ​		ctrl +t ---返回此前跳转位置
>
> ​		CTRL+o  在屏幕左边列出文件列表 
>
> ​		F4     列出函数列表

# 线程池

模块分析：

​				创建线程池：
