---
title: tinyhttpd源码注解
author: yanjinkai
categories:
  - web
  - http服务器
tags:
  - tinyhttpd
date: 2018-10-22 10:34:17
---
## tinyhttpd源码注解
### 网络通信预备知识
- 客户端-服务器通信模型
![客户端-服务器通信模型](https://box.kancloud.cn/2016-04-02_56ff80dbcb915.png)
### 头文件
c语言标准库头文件定义了一些通用变量，宏和函数
- <stdio.h> 
  - size_t: 无符号整数类型，它是 sizeof 关键字的结果
  - FILE: 适合存储文件流信息的对象类型
  - EOF: 表示到达文件结尾的负整数
  - NULL: 空指针常量
  - perror(const char* str): 将错误信息输出到stderr
  - FILE *fopen(const char *filename, const char *mode);//使用mode模式打开filename所指文件
  - int fclose(FILE *stream);//关闭文件流
  - int sprintf(char *str, const char *format, ...);//发送格式化输出到字符串
  - char *fgets(char *str, int n, FILE *stream);
- <stdlib.h>
  - void exit(int status);//终止当前进程，status表示退出状态（0代表正常退出）
  - int atoi(const char* str);//将字符串str转换为整数，如果不能有效转换返回0
  - int putenv(char *);
- <unistd.h>
  > On Unix-like systems, the interface defined by unistd.h is typically made up largely of system call wrapper functions such as fork, pipe and I/O primitives
  - int dup2(int, int);
  - int execl(const char *, const char *, ...);
  - ssize_t read(int, void *, size_t);
  - ssize_t write(int, const void *, size_t);
  - int close(int)
  - int pipe(int [2]);
  - pid_t fork(void);//对于子进程返回0，对于父进程返回子进程id
- <sys/socket.h>
  - int socket(int family, int type, int protocol);
    > 打开一个网络通讯端口，如果成功的话，就像open()一样返回一个文件描述符，应用程序可以像读写文件一样用read/write在网络上收发数据，如果socket()调用出错则返回-1。对于IPv4，family参数指定为AF_INET。对于TCP协议，type参数指定为SOCK_STREAM，表示面向流的传输协议。如果是UDP协议，则type参数指定为SOCK_DGRAM，表示面向数据报的传输协议。protocol参数的介绍从略，指定为0即可。
  - int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);
    > 将参数sockfd和myaddr绑定在一起，使sockfd这个用于网络通讯的文件描述符监听myaddr所描述的地址和端口号,myaddr参数实际上可以接受多种协议的sockaddr结构体，而它们的长度各不相同，所以需要第三个参数addrlen指定结构体的长度
  - int getsockname(int, struct sockaddr *restrict, socklen_t *restrict);
  - int listen(int sockfd, int backlog);
    > 声明sockfd处于监听状态，并且最多允许有backlog个客户端处于连接等待状态，如果接收到更多的连接请求就忽略
  - int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
    > cliaddr是一个传出参数，accept()返回时传出客户端的地址和端口号。addrlen参数是一个传入传出参数（value-result argument），传入的是调用者提供的缓冲区cliaddr的长度以避免缓冲区溢出问题，传出的是客户端地址结构体的实际长度（有可能没有占满调用者提供的缓冲区）。如果给cliaddr参数传NULL，表示不关心客户端的地址
  - ssize_t send(int socket, const void *buffer, size_t length, int flags);
  - ssize_t recv(int socket, void *buffer, size_t length, int flags);
    > 通常flags都设置为0，此时recv函数读取tcp buffer中的数据到buf中，并从tcp buffer中移除已读取的数据。把flags设置为MSG_PEEK，仅把tcp buffer中的数据读取到buf中，并不把已读取的数据从tcp buffer中移除，再次调用recv仍然可以读到刚才读到的数据
- <sys/stat.h>
  define stat structure
- <sys/wait.h>
  - pid_t waitpid(pid_t, int *, int);//等待子进程执行完
- <sys/types.h>
  - pid_t: Used for process IDs and process group IDs.
- <string.h>
  - void *memset(void *str, int c, size_t n);
  - char* stpcpy(char *restrict, const char *restrict);
  - int strcmp(const char *, const char *);
  - char* strcat(char *, const char *);
- <strings.h>
  - int strcasecmp(const char* str1, const char* str2);//忽略大小写比较字符串
- <ctype.h>
  - int isspace(int);
- <arpa/inet.h>
  下面四个函数用于网络字节序与主机字节序相互转换
  - uint32_t htonl(uint32_t);
  - uint16_t htons(uint16_t);
  - uint32_t ntohl(uint32_t);
  - uint16_t ntohs(uint16_t);
- <netinet/in.h>
  - sockaddr: 
    > In memory, the struct sockaddr_in and struct sockaddr_in6 share the same beginning structure as struct sockaddr, and you can freely cast the pointer of one type to the other without any harm, except the possible end of the universe.
  - sockaddr_in: ipv4 socket address type
### 主函数
调用startup，执行循环，循环体内调用accept接受请求（如果没有请求就阻塞）并调用accept_request处理请求
### startup
调用socket打开网络通讯端口，
调用memset,
调用bind绑定一个固定IP地址和端口,
如果传入的端口为0，调用getsockname随机分配端口，
调用listen,
### accept_request
调用get_line得到客户端发送的数据，解析出method,url,querystring,设置path,根据cgi标志量调用serve_file或者execute_cgi,关闭文件描述符
### serve_file
调用fopen读文件，如果不存在调用not_found,调用headers和cat,调用fclose关闭文件流
### execute_cgi
设置环境变量，pipe管道（实现父子进程通讯），dup2重定向标准输入输出到管道，fork子进程执行cgi程序
### get_line
调用recv读取一行数据并且将\r和\r\n替换为\n
### cat
调用fgets和send发送文档内容到客户端
### not_found
调用send发送404(not found)响应
### unimplemented
调用send发送501(method not implement)响应
### bad_request
调用send发送400(bad request)响应
### headers
调用send发送200(ok)响应头
### cannot_execute
调用send发送500(internal server error)响应
### error_die
调用perror和exit,输出错误并退出

[***源码链接***](https://sourceforge.net/projects/tinyhttpd/files/latest/download)