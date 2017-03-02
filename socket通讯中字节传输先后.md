---
title: socket通讯中字节传输先后
date: 2016-11-25 20:24:10
tags: Linuxc
categories: 编程学习
---

# socket通讯中字节传输先后

发送
```cpp
int main()
{
	int fd_c;
	struct sockaddr_in s_addr;
	fd_c = socket(AF_INET,SOCK_STREAM,0);
	s_addr.sin_family = AF_INET;
	s_addr.sin_port = htons(SERVER_PORT);
	inet_pton(AF_INET,SERVER_IP,&s_addr.sin_addr);
	if(-1 == connect(fd_c,(struct sockaddr *)&s_addr,sizeof(s_addr)))
	{
		perror("connect");
	}
	int a=0x12345678;
	//发送4个字节，
	write(fd_c,&a,sizeof(int));
	close(fd_c);
	return 0;
}
```

接收
```cpp
int main()
{

	int fd_s;
	uint16_t server_port = PORT;
	

	//creat addr about server
	struct sockaddr_in s_addr;
	struct sockaddr_in c_addr;
	int c_addr_size = 0;
	s_addr.sin_family = AF_INET;
	s_addr.sin_port = htons(server_port);
	s_addr.sin_addr.s_addr = htonl(INADDR_ANY);
	//create socket-->bind-->listen
	fd_s = socket(AF_INET,SOCK_STREAM,0);
	if(fd_s == -1)
	{
		perror("create server socket");
		return -1;
	}
	
	if(bind(fd_s,(struct sockaddr*)&s_addr,sizeof(s_addr))== -1)
	{
		perror("server bind");
		return -1;
	}
	if(-1 == listen(fd_s,N))
	{
		perror("server listen");
		return -1;
	}
	int fd_c = accept(fd_s,(struct sockaddr *)&c_addr,&c_addr_size);
	char a=0;
	//每次接收一个字节
	read(fd_c,&a,sizeof(a));
	printf("%x\n",(int)a);//78

	read(fd_c,&a,sizeof(a));
	printf("%x\n",(int)a);//56

	read(fd_c,&a,sizeof(a));
	printf("%x\n",(int)a);//34

	read(fd_c,&a,sizeof(a));
	printf("%x\n",(int)a);//12

	close(fd_s);
	close(fd_c);

}
```

结果分析：(前提客户端和服务器端都是小端)
1. 写入一个0x12345678
2. 读出，先读一个char，发送顺序从低位开始发送，每次发送一个char

联想：堆区栈区增长
```
int main()
{
    int a[2];
    printf("%x\n",&a[0]);
    printf("%x\n",&a[1]);
    int *b = malloc(sizeof(int)*2);
    printf("%x\n",&b[0]);
    printf("%x\n",&b[1]);
    return 0;

}
```
以上结果：
- a[1]地址比a[0]大4
- b[1]地址比b[0]大4

问题：为什么说堆区和栈区生长方式相反

```
int main()
{
    int a,b;
    printf("%x\n",&a);
    printf("%x\n",&b);
    int *c = malloc(sizeof(int));
    int *d = malloc(sizeof(int));
    printf("%x\n",c);
    printf("%x\n",d);

    return 0;

}
```

- 结果：b<a但是，d>c
- 分析
	- 堆区和栈区的增长体现在每次申请空间上
	- 栈区每次申请到的内存地址比上次的小
	- 堆区每次申请到的内存地址比上次的大