---
title: tcp相关概念
description: ""
date: 2023-02-26
tags:
  - 202302
  - tcp
categories:
  - tcp
menu: main
---

## 协议

ARP: ip换mac地址</br>
RARP: mac地址换ip

## socket

本质是内核维护的读写缓冲区形成的伪文件，与管道类似，管道是本地进程间通信，socket用于网络

## 字节序

<img src=/imgs/byte_sequence.png width=40%/></br>
网络字节序采用大端方式，主机根据需要自行转换

<!--more-->

```c
#include <stdio.h>

int main(){
  union {
    short value;
    char bytes[sizeof(short)];
  } test;

  test.value = 0x0102;
  if ((test.bytes[0] == 1) && (test.bytes[1] == 2)){
    printf("大端机器\n");
  } else if ((test.bytes[0] == 2) && (test.bytes[1] == 1)){
    printf("小端机器\n");
  }
  return 0;
}  
```

## socket接口

-|udp|tcp
--|--|--
是否创建连接|无连接|面向连接
是否可靠|不可靠|可靠
连接对象个数|一对一，一对多，多对一，多对多|一对一
传输方式|面向数据报|面向字节流
首部开销|8字节|最少20字节
适用场景|实时应用|可靠性高应用

<img src=/imgs/tcp_server_client.png width=40% />

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h> // 包含此文件头，可以不包含以上两个头

int socket(int domain, int type, int protocl);
// 功能: 创建一个套接字
// domain: 协议族
//  - AF_INET
//  - AF_INET6
//  - AF_UNIX
// type: 通信过程中使用的协议
//  - SOCK_STREAM 流式协议
//  - SOCK_DGRAM 报文式协议
// protocol: 具体的一个协议，一般写0
//  - SOCK_STREAM 默认使用TCP
//  - SOCK_DGRAM 默认使用UDP
// 返回值:
//  - 成功: 文件描述符，操作的是内核缓冲区
//  - 失败: -1

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// 功能: 绑定，将fd与本地的IP+端口进行绑定
// sockfd: 通过socket创建的描述符
// addr: 需要绑定的socket地址，这个地址封装了ip和port
// addrlen: 第二个参数结构体占的内存大小

int listen(int sockfd, int backlog);
// 功能: 监听需要与这个socket创建的连接
// sockfd: 通过socket创建的描述符
// backlog: 未连接与已连接和的最大值，超过此值会报拒接连接

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
// 功能: 接收客户端连接，默认是阻塞函数，阻塞等待客户端连接
// sockfd: 通过socket创建的描述符
// addr: 传出参数，记录连接成功hou客户端的地址信息(ip, port)
// addrlen: 第二个参数结构体占的内存大小
// 返回值:
//  - 成功: 用于通讯的文件描述符
//  - 失败: -1

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// 功能: 客户端连接服务器
// sockfd: 用于通讯的文件描述符
// addr: 客户端要链接的服务端的地址信息
// addrlen: 第二个参数结构体占的内存大小
// 返回值:
//  - 成功: 0
//  - 失败: -1

int write(int fd, const void *buf, size_t count);
int read(int fd, void *buf, size_t count);
```

server

```c
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>


int main(){
  // 创建套接字
  int sfd = socket(AF_INET, SOCK_STREAM, 0);
  if (sfd == -1){
    perror("socket");
    exit(-1);
  }

  // 绑定
  struct sockaddr_in addr;
  addr.sin_family = AF_INET;
  // inet_pton(AF_INET, "127.0.0.1", addr.sin_addr.s_addr);
  addr.sin_addr.s_addr = INADDR_ANY;
  addr.sin_port = htons(8765);
  int ret = bind(sfd, (const struct sockaddr *)&addr, sizeof(addr));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 监听
  ret = listen(sfd, 8);
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 等待客户端连接
  struct sockaddr_in client;
  socklen_t len = sizeof(client);
  int cfd = accept(sfd, (struct sockaddr *)&client, &len);
  if (cfd == -1){
    perror("accept");
    exit(-1);
  }

  // 输出客户端的信息
  char client_ip[16];
  inet_ntop(AF_INET, &client.sin_addr.s_addr, client_ip, sizeof(client_ip));
  unsigned short client_port; // 必须是无符号的
  client_port = ntohs(client.sin_port);

  printf("client ip: %s, client port: %d\n", client_ip, client_port);

  // 通讯
  char rbuff[1024]={0};
  int num = read(cfd, rbuff, sizeof(rbuff));
  if (num  == -1){
    perror("read");
    exit(-1);
  } else if (num > 0){
    printf("recv client data: %s\n", rbuff);
  } else{
    // 表示客户端断开连接
    printf("client close");
  }
                                                                                                                                 // 给客户端发送数据
  char *data = "hello, i am server";
  write(cfd, data, strlen(data));
  return 0;

  // 关闭文件描述符
  close(cfd);
  close(sfd);
}
```

client

```c
#include <arpa/inet.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <stdio.h>

int main(){
  // 创建套接字
  int cfd = socket(AF_INET, SOCK_STREAM, 0);
  if (cfd == -1){
    perror("socket");
    exit(-1);
  }

  // 连接服务端
  struct sockaddr_in serveraddr;
  serveraddr.sin_family = AF_INET;
  inet_pton(AF_INET, "127.0.0.1", &serveraddr.sin_addr.s_addr);
  // serveraddr.sin_addr.s_addr = INADDR_ANY;
  serveraddr.sin_port = htons(8765);
  int ret = connect(cfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));
  if (ret == -1){
    perror("connect");
    exit(-1);
  }

  // 通信
  // 给服务端发送数据
  char *data = "hello, i am client";
  write(cfd, data, strlen(data));
  char rbuff[1024]={0};
  int len = read(cfd, rbuff, sizeof(rbuff));
  if (len  == -1){
    perror("readerr");
    exit(-1);
  } else if (len > 0){
    printf("recv server data: %s\n", rbuff);
  } else{
    // 表示服务端断开连接
    printf("client close");
  }


  // 关闭文件描述符
  close(cfd);

  return 0;
} 
```

## tcp设计

{{< columns >}}

<img src=/imgs/tcp_head.png width=80%>


<--->

<img src=/imgs/three_handshake.png width=80%>

{{< /columns >}}

1. 第一次握手：
   1. 客户端将SYN标志位置为1
   2. 生成一个随机的32位的序号seq=J ， 这个序号后边是可以携带数据（数据的大小）
2. 第二次握手：
   1. 服务器端接收客户端的连接： ACK=1
   2. 服务器会回发一个确认序号： ack=客户端的序号 + 数据长度 + SYN/FIN(按一个字节算)
   3. 服务器端会向客户端发起连接请求： SYN=1
   4. 服务器会生成一个随机序号：seq = K
3. 第三次握手：
   1. 客户单应答服务器的连接请求：ACK=1
   2. 客户端回复收到了服务器端的数据：ack=服务端的序号 + 数据长度 + SYN/FIN(按一个字节算)

为什么需要三次握手，而不是两次

1. 可靠性: 确保双方收发能力正常
2. 安全性: 有效保证连接安全，防止SYN泛洪攻击，只有第三次ack的序列号正确时才被任务是正确的连接，而短时间内伪造正确的序列号比较困难，很可能超时被服务端的tcp协议栈中断连接

第三次握手可以携带数据

{{< columns >}}
滑动窗口

<img src=/imgs/slide_window.png width=100%/>

<--->

四次挥手
<img src=/imgs/four_shakehand.png width=100%>

{{< /columns >}}

## 多进程多线程服务器

client

```c
#include <netinet/in.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(){
  // 创建套接字
  int cfd = socket(AF_INET, SOCK_STREAM, 0);
  if (cfd == -1){
    perror("socket");
    exit(-1);
  }

  // 连接服务端
  struct sockaddr_in serveraddr;
  serveraddr.sin_family = AF_INET;
  inet_pton(AF_INET, "127.0.0.1", &serveraddr.sin_addr.s_addr);
  // serveraddr.sin_addr.s_addr = INADDR_ANY;
  serveraddr.sin_port = htons(8765);
  int ret = connect(cfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));
  if (ret == -1){
    perror("connect");
    exit(-1);
  }

  // 通信
  // 给服务端发送数据
  int i = 0;
  while (1){
    char rbuff[1024]={0};
    // char *data = "hello, i am client";
    sprintf(rbuff, "data: %d\n", i++);
    write(cfd, rbuff, strlen(rbuff));
    // sleep(1); // todo: 为何在这里Ctrl+C会导致服务器报reset by peer;
    int len = read(cfd, rbuff, sizeof(rbuff));
    if (len  == -1){
      perror("readerr");
      exit(-1);
    } else if (len > 0){
      printf("recv server data: %s\n", rbuff);
    } else{
      // 表示服务端断开连接
      printf("client close");
      break;
    }
    sleep(1);
  }
  // 关闭文件描述符
  close(cfd);

  return 0;
}

```

多进程

```c
#include <asm-generic/errno-base.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <signal.h>
#include <wait.h>
#include <errno.h>

void recyleChild(int arg){
  while(1){
    int ret = waitpid(-1, NULL, WNOHANG);
    if (ret == -1){
      // 所有子进程都回收了
      break;
    } else if (ret == 0){
      // 还有子进程活着;
      break;
    } else if (ret > 0) {
      // 被回收了
      printf("子进程 %d 被回收了\n", ret);
    }
  }
}


int main(){
  struct sigaction act;
  act.sa_flags = 0; // 第一个回调
  sigemptyset(&act.sa_mask);
  act.sa_handler = recyleChild;
  // 注册信号捕捉
  sigaction(SIGCHLD, &act, NULL);

  // 创建socket
  int sfd = socket(AF_INET, SOCK_STREAM, 0);
  if (sfd == -1){
    perror("socket");
    exit(-1);
  } 
  struct sockaddr_in saddr;
  saddr.sin_family = AF_INET;
  saddr.sin_port = htons(8765);
  saddr.sin_addr.s_addr = INADDR_ANY;

  // 绑定
  int ret = bind(sfd, (const struct sockaddr *)&saddr, sizeof(saddr));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 监听
  ret = listen(sfd, 8);
  if (ret == -1){
    perror("listen");
    exit(-1);
  }

  while(1){
    // 接收连接
    struct sockaddr_in caddr;
    socklen_t len = sizeof(caddr);
    int cfd = accept(sfd, (struct sockaddr *)&caddr, &len);
    if (cfd == -1){
      if (errno == EINTR){
        // 信号处理导致不阻塞产生的问题
        continue;
      }
      perror("accept");
      exit(-1);
    }

    // 每个连接创建一个子进程处理
    pid_t pid = fork();
    if (pid == 0){
      // 子进程
      
      // 获取客户端信息
      char cip[16];
      inet_ntop(AF_INET, &caddr.sin_addr.s_addr, cip, sizeof(cip));
      unsigned short cport = ntohs(caddr.sin_port);
      printf("clent ip: %s, client port: %d\n", cip, cport);


      char recv[1024];
      while (1){
        int num = read(cfd, recv, sizeof(recv));
        if (num == -1){
          perror("recv");
          exit(-1);
        } else if (num == 0){
          printf("client closed...");
          break;
        } else {
          printf("client [%s] say: %s\n", cip, recv);
        }
        char *ssay = "hello, you am server\n";
        write(cfd, ssay, strlen(ssay));
      }
      close(cfd);
      exit(0); // 退出当前子进程
    }
  }
  close(sfd);
}

```

多线程

```c
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <strings.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

struct sockInfo {
  int fd;
  pthread_t tid;
  struct sockaddr_in addr;
};

struct sockInfo sockinfos[128];

void * working(void *arg) {
  // 子线程和客户端通信 cfd, 客户端信息, 线程号
  struct sockInfo *sockinfo = (struct sockInfo *)arg;

  // 获取客户端信息
  char cip[16];
  inet_ntop(AF_INET, &sockinfo->addr.sin_addr.s_addr, cip, sizeof(cip));
  unsigned short cport = ntohs(sockinfo->addr.sin_port);
  printf("clent ip: %s, client port: %d\n", cip, cport);

  char recv[1024];
  while (1){
    int num = read(sockinfo->fd, recv, sizeof(recv));
    if (num == -1){
      perror("recv");
      exit(-1);
    } else if (num == 0){
      printf("client closed...");
      break;
    } else {
      printf("client [%s] say: %s\n", cip, recv);
    }
    char *ssay = "hello, you am server\n";
    write(sockinfo->fd, ssay, strlen(ssay));
  }
  close(sockinfo->fd);
  return NULL;
}

int main(){
  // 创建socket
  int sfd = socket(AF_INET, SOCK_STREAM, 0);
  if (sfd == -1){
    perror("socket");
    exit(-1);
  } 
  struct sockaddr_in saddr;
  saddr.sin_family = AF_INET;
  saddr.sin_port = htons(8765);
  saddr.sin_addr.s_addr = INADDR_ANY;

  // 绑定
  int ret = bind(sfd, (const struct sockaddr *)&saddr, sizeof(saddr));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 监听
  ret = listen(sfd, 8);
  if (ret == -1){
    perror("listen");
    exit(-1);
  }

  // 初始化数据
  int max = sizeof(sockinfos)/sizeof(sockinfos[0]);
  for(int i=0; i<max; i++){
    bzero(&sockinfos[i], sizeof(sockinfos[i]));
    sockinfos[i].fd = -1;
    sockinfos[i].tid = -1;
  }

  while(1){
    // 接收连接
    struct sockaddr_in caddr;
    socklen_t len = sizeof(caddr);
    int cfd = accept(sfd, (struct sockaddr *)&caddr, &len);
    if (cfd == -1){
      perror("accept");
      exit(-1);
    }

    // 每个连接创建一个子线程
    pthread_t tid;
    // struct sockInfo *info = malloc(sizeof(struct sockInfo)); // 管理内存需要安排释放的问题
    struct sockInfo *pinfo;
    for(int i=0; i<max; i++){
      // 从这个数组中d找到可用的sockinfo
      if (sockinfos[i].fd == -1){
        pinfo = &sockinfos[i];
        break;
      }
      if (i == max - 1) {
        sleep(1);
        i = 0;
      }
    }
    pinfo->fd = cfd;
    memcpy(&pinfo->addr, &caddr, sizeof(caddr));
    pthread_create(&pinfo->tid, NULL, working, pinfo);
    pthread_detach(pinfo->tid); // 子线程结束自己释放资源

  }
  close(sfd);
}
```

tcp状态转移

![tcp_state_transfer](/imgs/tcp_state_transfer.png)
![tcp_state_transfer2](/imgs/tcp_state_transfer2.png)

1. 在数据传输阶段没有状态转移
2. 2msl(max segment lifetime), 确认被动关闭方能收到最终的ack，被动关闭方总是会重传fin
3. 半关闭状态
4. 端口复用, sockopt函数设置

## select服务器

```c
#include <netinet/in.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <unistd.h>
#include <sys/select.h>
#include <string.h>
#include <stdlib.h>

int main(){
  int sfd = socket(AF_INET, SOCK_STREAM, 0);
  struct sockaddr_in saddr;
  saddr.sin_family = AF_INET;
  saddr.sin_addr.s_addr = INADDR_ANY;
  saddr.sin_port = htons(8765);

  // 绑定
  int ret = bind(sfd, (const struct sockaddr*)&saddr, sizeof(saddr));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 监听
  listen(sfd, 8);

  // 创建fd_set
  fd_set rdset, tmp;
  FD_ZERO(&rdset);
  FD_SET(sfd, &rdset);

  int maxfd = sfd;

  while (1) {
    tmp = rdset;
    // 调用select, 让内核帮检测哪些文件描述符有数据
    int ret = select(maxfd+1, &tmp, NULL, NULL, NULL);
    if (ret == -1){
      perror("select");
      exit(-1);
    } else if (ret == 0){
      continue;
    } else if (ret > 0) {
      // 说明检测到了文件描述符发生了改变
      if(FD_ISSET(sfd, &tmp)) {
        // 表示新的客户端连接进来了
        struct sockaddr_in caddr;
        socklen_t len = sizeof(caddr);
        int cfd = accept(sfd, (struct sockaddr *)&caddr, &len);
        // 将新的描述符加入集合中
        FD_SET(cfd, &rdset);
        // 更新最大描述符
        maxfd = cfd > maxfd ? cfd : maxfd;
      }

      for (int i=sfd+1; i < maxfd+1; i++){
        if(FD_ISSET(i, &tmp)){
          // 说明客户端发来了数据
          char buf[1024];
          int len = read(i, buf, sizeof(buf));
          if (len == -1){
            perror("read");
            exit(-1);
          } else if (len == 0){
            printf("client closed...\n");
            close(i);
            FD_CLR(i, &rdset);
          } else if (len > 0){
            printf("read buf: %s\n", buf);
            write(i, buf, strlen(buf));
          }
        }
      }
    }
  }
  close(sfd);
  FD_CLR(sfd, &rdset);
  return 0;
}
```

缺点:

1. 每次select都要把fd集合从用户态复制到内核态
2. 需要每次遍历fd集合,来确认设置位
3. select只支持1024个描述符
4. fds集合不能重用，每次都要重置

## poll服务器

```c
#include <netinet/in.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/poll.h>
#include <sys/socket.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <poll.h>

int main(){
  int sfd = socket(AF_INET, SOCK_STREAM, 0);
  struct sockaddr_in saddr;
  saddr.sin_family = AF_INET;
  saddr.sin_addr.s_addr = INADDR_ANY;
  saddr.sin_port = htons(8765);

  // 绑定
  int ret = bind(sfd, (const struct sockaddr*)&saddr, sizeof(saddr));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 监听
  listen(sfd, 8);

  // 初始化监测的描述符数组
  struct pollfd fds[1024];
  for (int i=0; i<1024; i++){
    fds[i].fd = -1;
    fds[i].events = POLLIN;
  }
  fds[0].fd = sfd;

  int nfds = 0;

  while (1) {
    // 调用poll, 让内核帮检测哪些文件描述符有数据
    int ret = poll(fds, nfds+1, -1);
    if (ret == -1){
      perror("select");
      exit(-1);
    } else if (ret == 0){
      continue;
    } else if (ret > 0) {
      // 说明检测到了文件描述符发生了改变
      if(fds[0].revents & POLLIN) {
        // 表示新的客户端连接进来了
        struct sockaddr_in caddr;
        socklen_t len = sizeof(caddr);
        int cfd = accept(sfd, (struct sockaddr *)&caddr, &len);
        // 将新的描述符加入集合中
        for (int i=0; i<1024; i++){
          if (fds[i].fd == -1){
            fds[i].fd = cfd;
            fds[i].events = POLLIN;
            break;
          }
        }
        // 更新最大描述符
        nfds = cfd > nfds ? cfd : nfds;
      }

      for (int i=1; i < nfds+1; i++){
        if(fds[i].revents & POLLIN){
          // 说明客户端发来了数据
          char buf[1024];
          int len = read(fds[i].fd, buf, sizeof(buf));
          if (len == -1){
            perror("read");
            exit(-1);
          } else if (len == 0){
            printf("client closed...\n");
            close(fds[i].fd);
            fds[i].fd = -1;
          } else if (len > 0){
            printf("read buf: %s\n", buf);
            write(fds[i].fd, buf, strlen(buf)+1);
          }
        }
      }
    }
  }
  close(sfd);
  return 0;
}
```

缺点:

1. 只解决了select的第三第四个问题

## epoll服务器

```c
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/epoll.h>
#include <arpa/inet.h>
#include <stdio.h>

int main(){
  int sfd = socket(AF_INET, SOCK_STREAM, 0);
  struct sockaddr_in saddr;
  saddr.sin_family = AF_INET;
  saddr.sin_addr.s_addr = INADDR_ANY;
  saddr.sin_port = htons(8765);

  // 绑定
  int ret = bind(sfd, (const struct sockaddr*)&saddr, sizeof(saddr));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 监听
  listen(sfd, 8);

  // 创建epoll实例
  int epfd = epoll_create(100);


  // 将监听fd加入epfd中;
  struct epoll_event epev;
  epev.events = EPOLLIN;
  epev.data.fd = sfd;
  epoll_ctl(epfd, EPOLL_CTL_ADD, sfd, &epev);

  struct epoll_event epevs[1024];
  while (1) {
    // 调用epoll_wait, 让内核帮检测哪些文件描述符有数据
    int ret = epoll_wait(epfd, epevs, 1024, -1);
    if (ret == -1){
      perror("select");
      exit(-1);
    } else if (ret == 0){
      continue;
    } else if (ret > 0) {
      // 说明检测到了文件描述符发生了改变
      for (int i=0; i<ret; i++){
        if (!(epevs[i].events & EPOLLIN)) {
          continue;
        }
        int curfd = epevs[i].data.fd;
        if ( curfd == sfd) {
          // 表示新的客户端连接进来了
          struct sockaddr_in caddr;
          socklen_t len = sizeof(caddr);
          int cfd = accept(sfd, (struct sockaddr *)&caddr, &len);
          // 将新的描述符加入ep中
          struct epoll_event epev;
          epev.events = EPOLLIN;
          epev.data.fd = cfd;
          epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &epev);
        } else {
          // 说明客户端发来了数据
          char buf[1024];
          int len = read(curfd, buf, sizeof(buf));
          if (len == -1){
            perror("read");
            exit(-1);
          } else if (len == 0){
            printf("client closed...\n");
            close(i);
            epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, &epev);
          } else if (len > 0){
            printf("read buf: %s\n", buf);
            write(curfd, buf, strlen(buf));
          }
        }
      }

    }
  }
  close(sfd);
  close(epfd);
  return 0;
}

```

epoll 两种工作模式 ET/LT

ET

```c
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <errno.h>

int main() {

    // 创建socket
    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in saddr;
    saddr.sin_port = htons(9999);
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = INADDR_ANY;

    // 绑定
    bind(lfd, (struct sockaddr *)&saddr, sizeof(saddr));

    // 监听
    listen(lfd, 8);

    // 调用epoll_create()创建一个epoll实例
    int epfd = epoll_create(100);

    // 将监听的文件描述符相关的检测信息添加到epoll实例中
    struct epoll_event epev;
    epev.events = EPOLLIN;
    epev.data.fd = lfd;
    epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &epev);

    struct epoll_event epevs[1024];

    while(1) {

        int ret = epoll_wait(epfd, epevs, 1024, -1);
        if(ret == -1) {
            perror("epoll_wait");
            exit(-1);
        }

        printf("ret = %d\n", ret);

        for(int i = 0; i < ret; i++) {

            int curfd = epevs[i].data.fd;

            if(curfd == lfd) {
                // 监听的文件描述符有数据达到，有客户端连接
                struct sockaddr_in cliaddr;
                int len = sizeof(cliaddr);
                int cfd = accept(lfd, (struct sockaddr *)&cliaddr, &len);

                // 设置cfd属性非阻塞
                int flag = fcntl(cfd, F_GETFL);
                flag |= O_NONBLOCK;
                fcntl(cfd, F_SETFL, flag);

                epev.events = EPOLLIN | EPOLLET;    // 设置边沿触发
                epev.data.fd = cfd;
                epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &epev);
            } else {
                if(epevs[i].events & EPOLLOUT) {
                    continue;
                }  

                // 循环读取出所有数据
                char buf[5];
                int len = 0;
                while( (len = read(curfd, buf, sizeof(buf))) > 0) {
                    // 打印数据
                    // printf("recv data : %s\n", buf);
                    write(STDOUT_FILENO, buf, len);
                    write(curfd, buf, len);
                }
                if(len == 0) {
                    printf("client closed....");
                }else if(len == -1) {
                    if(errno == EAGAIN) {
                        printf("data over.....");
                    }else {
                        perror("read");
                        exit(-1);
                    }
                    
                }

            }

        }
    }

    close(lfd);
    close(epfd);
    return 0;
}
```

LT

```c
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/epoll.h>

int main() {

    // 创建socket
    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in saddr;
    saddr.sin_port = htons(9999);
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = INADDR_ANY;

    // 绑定
    bind(lfd, (struct sockaddr *)&saddr, sizeof(saddr));

    // 监听
    listen(lfd, 8);

    // 调用epoll_create()创建一个epoll实例
    int epfd = epoll_create(100);

    // 将监听的文件描述符相关的检测信息添加到epoll实例中
    struct epoll_event epev;
    epev.events = EPOLLIN;
    epev.data.fd = lfd;
    epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &epev);

    struct epoll_event epevs[1024];

    while(1) {

        int ret = epoll_wait(epfd, epevs, 1024, -1);
        if(ret == -1) {
            perror("epoll_wait");
            exit(-1);
        }

        printf("ret = %d\n", ret);

        for(int i = 0; i < ret; i++) {

            int curfd = epevs[i].data.fd;

            if(curfd == lfd) {
                // 监听的文件描述符有数据达到，有客户端连接
                struct sockaddr_in cliaddr;
                int len = sizeof(cliaddr);
                int cfd = accept(lfd, (struct sockaddr *)&cliaddr, &len);

                epev.events = EPOLLIN;
                epev.data.fd = cfd;
                epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &epev);
            } else {
                if(epevs[i].events & EPOLLOUT) {
                    continue;
                }   
                // 有数据到达，需要通信
                char buf[5] = {0};
                int len = read(curfd, buf, sizeof(buf));
                if(len == -1) {
                    perror("read");
                    exit(-1);
                } else if(len == 0) {
                    printf("client closed...\n");
                    epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
                    close(curfd);
                } else if(len > 0) {
                    printf("read buf = %s\n", buf);
                    write(curfd, buf, strlen(buf) + 1);
                }

            }

        }
    }

    close(lfd);
    close(epfd);
    return 0;
}
```

## UDP

<img src=/imgs/udp.png width=50%>

server

```c
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <unistd.h>


int main(){
  int sfd = socket(AF_INET, SOCK_DGRAM, 0);
  if (sfd == -1){
    perror("socket");
    exit(-1);
  }

  // 绑定
  struct sockaddr_in sadd;
  sadd.sin_family = AF_INET;
  sadd.sin_port = htons(8765);
  sadd.sin_addr.s_addr = INADDR_ANY;

  int ret = bind(sfd, (const struct sockaddr*)&sadd, sizeof(sadd));
  if (ret == -1){
    perror("bind");
    exit(-1);
  }

  // 通信
  while (1) {
    // 接收数据
    struct sockaddr_in cadd;
    socklen_t len = sizeof(cadd);
    char buf[1024];
    int num =recvfrom(sfd, buf, sizeof(buf), 0, (struct sockaddr*)&cadd, &len);
    char ipbuf[16];
    printf("client ip:%s client port:%d\n",
           inet_ntop(AF_INET, &cadd.sin_addr.s_addr, ipbuf, sizeof(ipbuf)),
           ntohs(cadd.sin_port));
    printf("client say: %s\n", buf);

    // 发送数据
    sendto(sfd, buf, sizeof(buf)+1, 0, (struct sockaddr*)&cadd, sizeof(cadd));
  }

  close(sfd);
}

```

client

```c
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <unistd.h>


int main(){
  int sfd = socket(AF_INET, SOCK_DGRAM, 0);
  if (sfd == -1){
    perror("socket");
    exit(-1);
  }

  struct sockaddr_in sadd;
  sadd.sin_family = AF_INET;
  sadd.sin_port = htons(8765);
  inet_pton(AF_INET, "127.0.0.1", &sadd.sin_addr.s_addr);

  // 通信
  int i = 0;
  while (1) {
    char buf[1024];
    sprintf(buf, "hello, i am %d\n", i++);
    // 发送数据
    sendto(sfd, buf, sizeof(buf)+1, 0, (struct sockaddr*)&sadd, sizeof(sadd));

    // 接收数据
    socklen_t len = sizeof(sadd);
    int num =recvfrom(sfd, buf, sizeof(buf), 0, NULL, NULL);
    printf("server say: %s\n", buf);

    sleep(1);
  }

  close(sfd);
}

```

## 广播

1. 只能在局域网使用
2. 客户端需要绑定到服务端的广播端口，才能接收广播消息

server

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main() {

    // 1.创建一个通信的socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if(fd == -1) {
        perror("socket");
        exit(-1);
    }   

    // 2.设置广播属性
    int op = 1;
    setsockopt(fd, SOL_SOCKET, SO_BROADCAST, &op, sizeof(op));
    
    // 3.创建一个广播的地址
    struct sockaddr_in cliaddr;
    cliaddr.sin_family = AF_INET;
    cliaddr.sin_port = htons(9999);
    inet_pton(AF_INET, "192.168.193.255", &cliaddr.sin_addr.s_addr);

    // 3.通信
    int num = 0;
    while(1) {
       
        char sendBuf[128];
        sprintf(sendBuf, "hello, client....%d\n", num++);
        // 发送数据
        sendto(fd, sendBuf, strlen(sendBuf) + 1, 0, (struct sockaddr *)&cliaddr, sizeof(cliaddr));
        printf("广播的数据：%s\n", sendBuf);
        sleep(1);
    }

    close(fd);
    return 0;
}
```

client

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main() {

    // 1.创建一个通信的socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if(fd == -1) {
        perror("socket");
        exit(-1);
    }   

    struct in_addr in;

    // 2.客户端绑定本地的IP和端口
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    addr.sin_addr.s_addr = INADDR_ANY;

    int ret = bind(fd, (struct sockaddr *)&addr, sizeof(addr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3.通信
    while(1) {
        
        char buf[128];
        // 接收数据
        int num = recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("server say : %s\n", buf);

    }

    close(fd);
    return 0;
}
```

## 组播(多播)

1. 组播既可以用于局域网，也可以用于广域网
2. 客户端需要加入多播组，才能接收到多播的数据

server

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main() {

    // 1.创建一个通信的socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if(fd == -1) {
        perror("socket");
        exit(-1);
    }   

    // 2.设置多播的属性，设置外出接口
    struct in_addr imr_multiaddr;
    // 初始化多播地址
    inet_pton(AF_INET, "239.0.0.10", &imr_multiaddr.s_addr);
    setsockopt(fd, IPPROTO_IP, IP_MULTICAST_IF, &imr_multiaddr, sizeof(imr_multiaddr));
    
    // 3.初始化客户端的地址信息
    struct sockaddr_in cliaddr;
    cliaddr.sin_family = AF_INET;
    cliaddr.sin_port = htons(9999);
    inet_pton(AF_INET, "239.0.0.10", &cliaddr.sin_addr.s_addr);

    // 3.通信
    int num = 0;
    while(1) {
       
        char sendBuf[128];
        sprintf(sendBuf, "hello, client....%d\n", num++);
        // 发送数据
        sendto(fd, sendBuf, strlen(sendBuf) + 1, 0, (struct sockaddr *)&cliaddr, sizeof(cliaddr));
        printf("组播的数据：%s\n", sendBuf);
        sleep(1);
    }

    close(fd);
    return 0;
}
```

client

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main() {

    // 1.创建一个通信的socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if(fd == -1) {
        perror("socket");
        exit(-1);
    }   

    struct in_addr in;
    // 2.客户端绑定本地的IP和端口
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    addr.sin_addr.s_addr = INADDR_ANY;

    int ret = bind(fd, (struct sockaddr *)&addr, sizeof(addr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    struct ip_mreq op;
    inet_pton(AF_INET, "239.0.0.10", &op.imr_multiaddr.s_addr);
    op.imr_interface.s_addr = INADDR_ANY;

    // 加入到多播组
    setsockopt(fd, IPPROTO_IP, IP_ADD_MEMBERSHIP, &op, sizeof(op));

    // 3.通信
    while(1) {
        
        char buf[128];
        // 接收数据
        int num = recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("server say : %s\n", buf);

    }

    close(fd);
    return 0;
}
```

## 本地域套接字

server

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/un.h>

int main() {

    unlink("server.sock");

    // 1.创建监听的套接字
    int lfd = socket(AF_LOCAL, SOCK_STREAM, 0);
    if(lfd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2.绑定本地套接字文件
    struct sockaddr_un addr;
    addr.sun_family = AF_LOCAL;
    strcpy(addr.sun_path, "server.sock");
    int ret = bind(lfd, (struct sockaddr *)&addr, sizeof(addr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3.监听
    ret = listen(lfd, 100);
    if(ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 4.等待客户端连接
    struct sockaddr_un cliaddr;
    int len = sizeof(cliaddr);
    int cfd = accept(lfd, (struct sockaddr *)&cliaddr, &len);
    if(cfd == -1) {
        perror("accept");
        exit(-1);
    }

    printf("client socket filename: %s\n", cliaddr.sun_path);

    // 5.通信
    while(1) {

        char buf[128];
        int len = recv(cfd, buf, sizeof(buf), 0);

        if(len == -1) {
            perror("recv");
            exit(-1);
        } else if(len == 0) {
            printf("client closed....\n");
            break;
        } else if(len > 0) {
            printf("client say : %s\n", buf);
            send(cfd, buf, len, 0);
        }

    }

    close(cfd);
    close(lfd);

    return 0;
}
```

client

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/un.h>

int main() {

    unlink("client.sock");

    // 1.创建套接字
    int cfd = socket(AF_LOCAL, SOCK_STREAM, 0);
    if(cfd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2.绑定本地套接字文件
    struct sockaddr_un addr;
    addr.sun_family = AF_LOCAL;
    strcpy(addr.sun_path, "client.sock");
    int ret = bind(cfd, (struct sockaddr *)&addr, sizeof(addr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3.连接服务器
    struct sockaddr_un seraddr;
    seraddr.sun_family = AF_LOCAL;
    strcpy(seraddr.sun_path, "server.sock");
    ret = connect(cfd, (struct sockaddr *)&seraddr, sizeof(seraddr));
    if(ret == -1) {
        perror("connect");
        exit(-1);
    }

    // 4.通信
    int num = 0;
    while(1) {

        // 发送数据
        char buf[128];
        sprintf(buf, "hello, i am client %d\n", num++);
        send(cfd, buf, strlen(buf) + 1, 0);
        printf("client say : %s\n", buf);

        // 接收数据
        int len = recv(cfd, buf, sizeof(buf), 0);

        if(len == -1) {
            perror("recv");
            exit(-1);
        } else if(len == 0) {
            printf("server closed....\n");
            break;
        } else if(len > 0) {
            printf("server say : %s\n", buf);
        }

        sleep(1);

    }

    close(cfd);
    return 0;
}
```
