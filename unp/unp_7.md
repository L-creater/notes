# 第七章 套接字选项

## 7.2 getssockopt 和 setsockopt 函数

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int optname,
                      void *optval, socklen_t *optlen);

	sockfd:必须指向一个打开的套接字描述符;
	optname:
	levle: ;
	optval:getsockopt以获取的选项当前值存放到 *optval;
	optlen:optval长度
   
int setsockopt(int sockfd, int level, int optname,
                      const void *optval, socklen_t optlen);

```

```c
/*
getsockname根据accpet返回的sockfd，得到临时端口号
getpeername根据accpet返回的sockfd，得到远端链接的端口号，在exec后可以获取客户端信息。
*/
//getsockname获取sockfd对应的本端socket地址，并将其存储于address参数指定的内存中，
//该socket地址的长度则存储于address_len参数指向的变量中。
//实际socket地址的长度大于address所指内存的区的大小，则该socket地址将被截断。
getpeername获取sockfd对应的远端sockfd
```

## 7.5 通用套接字选项

### 7.5.1 SO_BROADCAST 套接字选项

本选项开启或禁止发送广播消息的能力，只有数据报套接字支持广播，并且还必须是在支持广播消息的网络上。

### 7.5.2 SO_DEBUG 套接字选项

本选项仅由TCP支持。当给一个套接字开启本选项时，内核将为TCP在该套接字发送和接受的所有分组保留详细跟踪信息。这些信息保存在内核的某个环形缓冲区，并可用trpt程序进行检查。

### 7.5.3 SO_DONTROUTE 套接字选项

不查看路由表，直接将数据发送给本地局域网内的主机。

### 7.5.4 SO_ERROR 套接字选项

获取并清除socket错误状态。

### 7.5.5 S0_KEEPPALIVE 套接字选项

发送周期性保活报文以维持连接。

### 7.5.6 SO_LINGER 套接字选项

若有数据待发送，则延迟关闭。

### 7.5.7 SO_OOBINLINE 套接字选项

开启时，接收到的带外数据将存留在普通数据的输入队列中，此时我们不能使用MSG_OOB标志的读操作来读取带外数据（而应该像读取普通数据那样读取带外数据）

### 7.5.8 SO_RCVBUF 和 SO_SNDBUF 套接字选项

分别表示TCP接收缓冲区和发送缓冲区的大小。

套接字缓冲区大小总是由新创建的已连接套接字从监听套接字继承而来。

### 7.5.9 SO_RCVLOWAT 和 SO_SNDLOWAT 

SO_RCVLOWAT 和 SO_SNDLOWAT  分别表示TCP接收缓冲区和发送缓冲区的低水位标记。它们一般被IO复用系统调用 用来判断socket是否可读或可写，当TCP接收缓冲区中可读数据总数大于其低水位标记时，io服用将通知应用程序可以从对应的socket上读取数据；当TCP发送缓冲区的空闲时间（可写入数据的空间）大于其低水位标记时，IO复用系统调用通知应用程序可以往对应的socket上写入数据。

### 7.5.10  SO_RCVTIMEO 和 SO_SNDTIMEO 套接字选项

给套接字的接收和发送设置一个超时值。

### 7.5.11 SO_REUSEADDR 和 SO_REUSEPORT 





### 7.5.12 SO_TYPE 

获取socket类型，返回的整数值是SOCK_STREAM或SOCK_DGRAM之类的值。

### 7.5.13 SO_USELOOPBACK	

## 7.11 fcnl 函数

```c
#include <unistd.h>
#include <fcntl.h>
//可执行各种描述符控制操作。

int fcntl(int fd, int cmd, ... /* arg */ );




```



#   第八章 基本UDP套接字编程

## 8.2 resvfrom 和 sendto 函数

```c
实现C/S模型：
    recv()/send()只能用于TCP通信。替代read/write
    accept();---connect; ---被舍弃
    server:
			lfd = socket(AF_INET,SOCK_DGRAM,0);  //SOCK_DGRAM---报式协议
			bind();
			listen();//---可有可无
			while(1)
            {
                read(,buf,sizeof(buf));//---被替换 --recvfrom()--涵盖accept传出地址结构
                ssize_t recvfrom(int socket,void *buf,size_t len, int flags, struct sockaddr *src_addr,sockle_t *addrlen);
                socket:套接字;
                buf:缓冲地址;
                len:缓冲区大小;
                flags:0;
                src_addr:(struct sockaddr *)&addr 传出。对端地址结构;
                addrlen: 传入传出。（值-结果参数）
                    
                return: 成功接收字节数，失败 -1，erron ，0:对端关闭;
                
                
                write();//----被替换为sendto()
                ssize_t sendto(int socket,const void *buf,size_t len, int flags, struct sockaddr *src_addr,sockle_t *addrlen);
                socket:lfd;
                buf:存储数据的缓冲地址;
                len:数据长度;
                flags:0;
                src_addr:(struct sockaddr *)&addr 传入。目标地址结构;
                addrlen: 地址结构长度。
                    
                return: 成功写出字节数，失败 -1，erron 
            }
            
	client:
			connfd = socket(AF_INET,SOCK_DGRAM,0); 
			sendto('服务器地址结构'，"地址结构大小");//--connect(connfd,"服务器地址结构"，地址结构大小);
			
			
			recvfrom();
			//写到屏幕
			close();
   
```

server:

```c
#include <string.h>
#include <netinet/in.h>
#include <stdio.h>
#include <unistd.h>
#include <strings.h>
#include <arpa/inet.h>
#include <ctype.h>

#define MAXLINE 80
#define SERV_PORT 6666

int main(void)
{
	struct sockaddr_in servaddr, cliaddr;
	socklen_t cliaddr_len;
	int sockfd;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int i, n;

	sockfd = socket(AF_INET, SOCK_DGRAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	bind(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
	printf("Accepting connections ...\n");

	while (1) {
		cliaddr_len = sizeof(cliaddr);
		n = recvfrom(sockfd, buf, MAXLINE,0, (struct sockaddr *)&cliaddr, &cliaddr_len);
		if (n == -1)
			perror("recvfrom error");
		printf("received from %s at PORT %d\n", 
				inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
				ntohs(cliaddr.sin_port));
		for (i = 0; i < n; i++)
			buf[i] = toupper(buf[i]);

		n = sendto(sockfd, buf, n, 0, (struct sockaddr *)&cliaddr, sizeof(cliaddr));
		if (n == -1)
			perror("sendto error");
	}
	close(sockfd);
	return 0;
}

```

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <strings.h>
#include <ctype.h>

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	int sockfd, n;
	char buf[MAXLINE];

	sockfd = socket(AF_INET, SOCK_DGRAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		n = sendto(sockfd, buf, strlen(buf), 0, (struct sockaddr *)&servaddr, sizeof(servaddr));
		if (n == -1)
			perror("sendto error");
		n = recvfrom(sockfd, buf, MAXLINE, 0, NULL, 0);
		if (n == -1)
			perror("recvfrom error");
		write(STDOUT_FILENO, buf, n);
	}
	close(sockfd);
	return 0;
}

```

