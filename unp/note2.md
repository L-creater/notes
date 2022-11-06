

```c
#include <sys/socket.h>
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
	int n = 220x1024
	setsockopt(sockfd, SOL_SOCKET, SO_RCVBUF, &n, sizeof(n));

```



# 线程池模块分析：

​					1.main();

​									创建线程池。

​									向线程池中添加任务。借助回调处理任务

​									销毁线程池

​						2.pthreadpool_creat();

​									创建线程池结构体 指针

​									初始化线程池结构体

​									创建N个任务线程。

​									 失败时：销毁开辟的所有空间。(释放)

TCP和udp通信各自优缺点：

​		TCP:		面向连接的，可靠数据报传输。		对于不稳定的网络层，采取完全						 弥补的通信方式---丢包重传。

​						 优点：稳定。 数据流量稳定，速度稳定，传递的顺序

​						 缺点：传输速度慢，效率低，系统资源开销大

​						 使用场景：数据的完整性要求较高，不追求效率。大数据传输、文件传输

​		udp:		无连接，不可靠的数据报传输。		对于不稳定的网络层，采取完全						 弥补的通信方式---默认还原网络状况。

​						 优点：传输速度快，效率高，系统资源开销低

​						 缺点：不稳定，数据流量、速度、顺序。

​						 使用场景：时效性要求较高，稳定性其次。  游戏、视频会议、视频电话。

# UDP

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
                addrlen: 传入传出。
                    
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

> man recvfrom 
>
> man sendto 



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
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, n;
char *str;

	if (argc != 2) {
		fputs("usage: ./client message\n", stderr);
		exit(1);
	}
str = argv[1];

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	write(sockfd, str, strlen(str));

	n = read(sockfd, buf, MAXLINE);
	printf("Response from server:\n");
	write(STDOUT_FILENO, buf, n);
	close(sockfd);

	return 0;
}

```



# libevent

​			开源，精简，跨平台，专注于网络通信。



```c
libevent 框架：

1. 创建 event_base		底座

struct event_base	*event_base_new(void);

strucr event_base	*base = event_base_new();

2. 创建事件 event
					
常规事件 event--->event_new();

带缓冲区bufferevent---bufferevent_socket_new();
					

3. 将事件添加到base上

int event_add(struct event *ev, const struct timeval *tv);

4. 循环监听事件满足

int event_base_dispatch(struct event_base *base);
									event_base_dispatch();

5. 释放 event_base
							event_base_free(base);



特性：
		基于“事件”异步通信模型。
```

```c
创建事件:
	struct event *event_new(struct event_base *base，evutil_socket_t fd，short what，event_callback_fn cb;  void *arg);

base： event_base_new()返回值。

		 fd： 绑定到 event 上的 文件描述符

		what：对应的事件（r、w、e）

			EV_READ		一次 读事件

			EV_WRTIE	一次 写事件

			EV_PERSIST	持续触发。 结合 event_base_dispatch 函数使用，生效。

		cb：一旦事件满足监听条件，回调的函数。

		typedef void (*event_callback_fn)(evutil_socket_t fd,  short,  void *)	

		arg： 回调的函数的参数。

		返回值：成功创建的 event


```

```c
添加事件:
		添加事件到 event_base

	int event_add(struct event *ev, const struct timeval *tv);

		ev: event_new() 的返回值。

		tv：NULL
            
从event_base上摘下事件				【了解】

	int event_del(struct event *ev);

		ev: event_new() 的返回值。
```

```c
释放事件:	
	int event_free(struct event *ev);

		ev: event_new() 的返回值。
```

非未决：没有资格被处理

未决：有资格被处理，但还未处理

event_new -->event--->非未决--->event_add--->未决--->diapatch()--->执行回调函数--->处理态--->非未决 event_add && EV_PERSIST--->未决--->event_del--->非未决

## bufferevent

```c
bufferevent:	//
		带缓冲区的事件 bufferevent;


#include <event2/bufferevent.h> 

	read/write 两个缓冲. 借助 队列.


创建bufferevent：

	struct bufferevent *ev；

	struct bufferevent *bufferevent_socket_new(struct event_base *base, evutil_socket_t fd, enum bufferevent_options options);

		base： event_base

		fd:	封装到bufferevent内的 fd

		options：BEV_OPT_CLOSE_ON_FREE---释放bufferevent时关闭底层传输端口。这将关闭底层套接字，释放底层bufferevent等

	返回： 成功创建的 bufferevent事件对象。

            
            
销毁bufferevent：	
	void  bufferevent_socket_free(struct bufferevent *ev);


    
```

