---
title: socket和epoll
date: 2017-03-03 15:51:27
tags: Linuxc
categories: 编程学习
---
# socket以及epoll 

## socket

前期：struct sockaddr_in serveraddr,cliaddr;

server端：
```
1. listenfd = socket(AF_INET,SOCK_STREAM,0);//AF_INET:ipv4，SOCK_STREAM：流式协议
2.  bind(listenfd,(struct sockaddr *)&serveraddr,sizeof(serveraddr));
	- bzero(&serveraddr,sizeof(serveraddr));
	- serveraddr.sin_family = AF_INET;
    - serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
    - serveraddr.sin_port = htons(8880);
3.  listen(listenfd,20);
4.  connfd = accept(listenfd,(struct sockaddr*)&cliaddr,&cliaddr_len);//cliaddr_len = sizeof(cliaddr)
	- printf("new conn:ip %s,port:%d\n",inet_ntoa(cliaddr.sin_addr),ntohs(cliaddr.sin_port));
```

client端：
```
1. sockfd = socket(AF_INET,SOCK_STREAM,0);
2. connect(sockfd,(struct sockaddr*)&serveraddr,sizeof(serveraddr));
```
 
## epoll常用函数

注意：
1. 每次从fd中读取数据时将buf清零
2. EPOLLIN事件，当客户端关闭时从fd中读取的数据长度为0此时记得关闭fd
	- serveraddr.sin_family = AF_INET;
    - inet_pton(AF_INET,"127.0.0.1",&serveraddr.sin_addr);
    - serveraddr.sin_port = htons(8880);

```cpp
/**
 * @file server.c
 * @brief  epoll函数的服务器端
 * @author chibaobao
 * @version 
 * @date 2017-01-18
 */
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/epoll.h>

int main()
{
    struct sockaddr_in serveraddr,cliaddr;
    int cliaddr_len = sizeof(cliaddr);
    int listenfd,connfd;
    char buf[1024];
    char str[INET_ADDRSTRLEN];
    int i=0;
    int efd;
    listenfd = socket(AF_INET,SOCK_STREAM,0);
    bzero(&serveraddr,sizeof(serveraddr));
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
    serveraddr.sin_port = htons(8880);

    bind(listenfd,(struct sockaddr *)&serveraddr,sizeof(serveraddr));
    listen(listenfd,20);
    
    struct epoll_event event;
    struct epoll_event list_event[1024];
    int res,len;
    efd = epoll_create(10);
    
    event.events = EPOLLIN;

    event.data.fd=listenfd;
    epoll_ctl(efd,EPOLL_CTL_ADD,listenfd,&event);


    
    while(1)
    {
        res = epoll_wait(efd,list_event,1024,-1);
        for(i=0;i<res;i++)
        {
            if(list_event[i].data.fd == listenfd)
            {
                connfd = accept(listenfd,(struct sockaddr*)&cliaddr,&cliaddr_len);
                printf("new conn:ip %s,port:%d\n",inet_ntoa(cliaddr.sin_addr),ntohs(cliaddr.sin_port));
                event.data.fd=connfd;
                epoll_ctl(efd,EPOLL_CTL_ADD,connfd,&event);
            }
            else if(list_event[i].events & EPOLLIN)
            {
                bzero(buf,sizeof(buf));
                len = read(list_event[i].data.fd,buf,sizeof(buf));
                if(len == 0)
                {
                    epoll_ctl(efd, EPOLL_CTL_DEL, list_event[i].data.fd, NULL);
                    close(list_event[i].data.fd);
                }
                printf("str[%d]:#%s#\n",i,buf);
            }
        }
    }
}

```

client

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
int main()
{
    struct sockaddr_in serveraddr;
    char buf[1024];
    int sockfd,i;
    bzero(buf,sizeof(buf));
    bzero(&serveraddr,sizeof(serveraddr));

    sockfd = socket(AF_INET,SOCK_STREAM,0);
    serveraddr.sin_family = AF_INET;
    inet_pton(AF_INET,"127.0.0.1",&serveraddr.sin_addr);
    serveraddr.sin_port = htons(8880);

    connect(sockfd,(struct sockaddr*)&serveraddr,sizeof(serveraddr));
    while(1)
    {
        bzero(buf,sizeof(buf));
        strcpy(buf,"aaabbbccc\n");
        write(sockfd,buf,strlen(buf));
        sleep(3);
    }
    close(sockfd);
}
```