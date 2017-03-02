---
title: linux高级编程-多进程
date: 2016-10-13 23:11:24
tags: Linuxc
categories: 编程学习
---

# 多进程

- 进程状态: 就绪、运行、挂起、停止等状态
- 父子进程共享数据:  fork之后，子进程会拷贝父进程的数据空间、堆和栈空间（实际上是采用写时复制技术），二者共享代码段。
- 多进程一定要记得wait回收


## 多进程相关函数

- fork()
	- 返回值:为0表示当前是子进程,大于0表示当前是父进程并且返回值为子进程pid
	
	```
	int b=10;
	int main()
	{
	    int pid;
	    int a=10;
	    int *c=(int*)malloc(sizeof(int));
	    *c=10;
	    pid = fork();
	    if(pid == 0)
	    {   
	        a=100;
	        b=100;
	        *c=100;
	        printf("child:a=%d,b=%d,c=%d\n",a,b,*c);//结果为100,100,100
	    }   
	    else
	    {   
	        a=101;
	        b=101;
	        *c=101;
	        printf("parent:a=%d,b=%d,c=%d\n",a,b,*c);//结果为101,101,101
	    }   
	    return 0;
	}
	```
- getpid()
	- 获取当前进程进程ID
- getppid()
	- 获取当前进程的父进程ID
- getuid() getgid()
	- 获取当前进程uid和gid

## exec函数族
**注意**：arg最后一个传NULL

```cpp
eg:execlp("echo","echo","hello", NULL);
//arg 分别包括命令，参数，NULL

execlp("echo","hello", NULL);//是错误的
```

```
int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg, ..., char *const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execve(const char *path, char *const argv[], char *const envp[]);
```

## 子进程回收

### 孤儿进程与僵尸进程
- 孤儿进程: 父进程先于子进程结束，则子进程成为孤儿进程，子进程的父进程成为init进程，称为init进程领养孤儿进程。
- 僵尸进程: 进程终止，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成僵尸（Zombie）进程。

### wait相关函数

- pid_t wait(int *status);
	- 功能:阻塞,回收一个子进程
	- 返回值:成功：清理掉的子进程ID；失败：-1 (没有子进程)
	- 参数:传出参数(宏)

	 	1.  WIFEXITED(status) 为非0	→ 进程正常结束
			WEXITSTATUS(status) 如上宏为真，使用此宏 → 获取进程退出状态 (exit的参数)
		2.  WIFSIGNALED(status) 为非0 → 进程异常终止
		  WTERMSIG(status) 如上宏为真，使用此宏 → 取得使进程终止的那个信号的编号。
		3.  WIFSTOPPED(status) 为非0 → 进程处于暂停状态
		WSTOPSIG(status) 如上宏为真，使用此宏 → 取得使进程暂停的那个信号的编号。
		WIFCONTINUED(status) 为真 → 进程暂停后已经继续运行

- pid_t waitpid(pid_t pid, int *status, in options);
	- 功能:
	- 返回值:	成功：返回清理掉的子进程ID；失败：-1(无子进程)
	- 参数:
	    - pid
		
		    - > 0 回收指定ID的子进程	
		    - -1 回收任意子进程（相当于wait）
		    - 0 回收和当前调用waitpid一个组的所有子进程
		    - < -1 回收指定进程组内的任意子进程

		- status 	
		    - 与wait相同
		
		- options
		    - WNOHANG 不阻塞

	```
	 while(1)
     {
            pid = waitpid(0,NULL,WNOHANG);//WNOHANG不阻塞,三个返回值 0代表当前无子进程退出,>0代表有子进程退出,-1代表所有子进程意见回收完毕
            if(pid > 0)
                printf("relase pid %d\n",pid);
            if(pid < 0)
                break;         
            sleep(1);
    }
	```
	
##	进程间通信

1. 管道 (使用最简单)
2. 信号 (开销最小)
3. 共享内存 (无血缘关系)
4. 本地套接字 (最稳定)

### 管道

#### 管道(pipe)
只能父子进程通信使用
- int pipe(int pipefd[2]);
	- 成功：0；失败：-1，设置errno
- 使用方法
	1. int fd[2];//创建pipe文件描述符数组
	2. pipe(fd);//初始化fd为两个管道fd[0]为读端,fd[1]为写端
		- 父/子进程  关闭读端,向写端写数据
		- 子/父进程  关闭写端,从读端读数据
 
	```
	int main()
	{
	    printf("begin\n");
	    int fd[2];
	    if(pipe(fd) == -1)//创建管道,fd[2]会被初始化,之后可以直接使用
	    {
	        perror("pipe err");
	        exit(1);
	    }
	    pid_t pid = fork();
	    if(pid == -1)
	    {
	        perror("fork err:");
	        exit(1);
	    }
	    else if(pid > 0)
	    {
	        close(fd[1]);//父进程关闭写端
	        dup2(fd[0],STDIN_FILENO);//将管道读端拷贝给标准输入
	        execlp("wc","wc","-l",NULL);//执行wc
	        close(fd[0]);
	    }
	    else
	    {
	        close(fd[0]);//fd[0]代表读端,子进程关闭读端
	        dup2(fd[1],STDOUT_FILENO);//将管道写端拷贝给标准输出
	        execlp("ls","ls",NULL);
	        close(fd[1]);
	    }
	    return 0;
	}
	```

#### 管道(fifo)

- int mkfifo(const char *pathname,  mode_t mode);  成功：0； 失败：-1
- 使用方法:
	1. $   mkfifo 管道名 
		创建管道文件
	2. 特点:
		- 没有读端读取,写端阻塞 	
		- 不可重复读取(读走了数据数据就没有了), ***fifo文件实现方式是队列***
	3. 	代码:
		1. 	int fd[2];
		2. 	
		写端
		
		```
		int main(int argc, char *argv[])
		{
		    int fd, i;
		    char buf[4096];
		    fd = open("fifo_file", O_WRONLY);
		    sprintf(buf, "hello itcast\n");
		    write(fd, buf, strlen(buf));//没有读端读取就会阻塞
		    //sleep(5);
		    close(fd);
		
		    return 0;
		}
		```
		读端
		```
		int main(int argc, char *argv[])
		{
		    int fd, len;
		    char buf[4096];
		    fd = open("fifo_file", O_RDONLY);
		    //sleep(3);
		    len = read(fd, buf, sizeof(buf));
		    write(STDOUT_FILENO, buf, len);
		    close(fd);    
		    return 0;
		}
		```

### 信号

$ kill -l查看信号

- 注册信号signal
	- sighandler_t signal(int signum, sighandler_t handler);
	- signum信号编号（一般用信号的宏）
	- handler 函数指针（函数为void fun(int signum)格式的回调函数）

- 发送信号kill(pid,sig);发送信号给pid进程，
	- pid>0进程号，pid=0同组进程，pid=-1所有可送达的进程，pid<-1组id绝对值。

```
信号相关各种操作（拓展）
信号：（名称，编号，处理动作，发生条件）--->软中断
	
信号
	pending----> | mask 阻塞(置1后时pengding被置1的位无法送达)  
	产生--->未决  |---->递达
				 |
				
	默认处理动作：	term：终止进程
					core：终止进程，并产生core文件
					ign ：忽略
					stop：暂停
					cont：继续
kill(pid,sig);发送信号给pid进程，pid>0进程号，pid=0同组进程，pid=-1所有可送达的进程，pid<-1组id绝对值，
raise(sig);	向本进程发送sig


alarm(time)//设置时钟返回上次时钟剩余时间，time秒后发送SIGALRM
int setitimer(int which, const struct itimerval *new_value,struct itimerval *old_value)//
struct itimerval {
					struct timeval it_interval; /* next value */
					struct timeval it_value;    /* current value */
				};

struct timeval {
					time_t      tv_sec;         /* seconds */
					suseconds_t tv_usec;        /* microseconds */
				};

				
				
信号操作函数：
信号集 set
			
			sigset_t   set;

			sigemptyset（&set）  清空集合  

			sigfillset（&set）  置1集合  

			sigaddset（&set， 待添加的信号名称） 添加信号到集合中

			sigdelset（&set， 待删除的信号名称） 删除信号到集合中

			sigismember（&set， 待判断的信号名称）判断信号是否在集合中  -- 》 1：在 0：不在； -1；错
mask操作

			  int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

				参1： 	SIG_BLOCK  将set中的信号添加到mask屏蔽字中

				 	SIG_UNBLOCK  将set中的信号从mask屏蔽字中接触屏蔽

					SIG_SETMASK  用set覆盖mask

				参2：  用来操作mask的set集合

				参3：	记录旧有mask的状态。

pending操作

			int sigpending(sigset_t *set);

			参数： 传出的未决信号集

			返回值：成0失败-1 errno


signal(sig,funtion)//sig信号绑定funtion函数
int sigaction(int signum, const struct sigaction *act,struct sigaction *oldact);
struct sigaction {
					void     (*sa_handler)(int);//要绑定的函数,通过宏，可以设置执行默认动作或者忽略
					void     (*sa_sigaction)(int, siginfo_t *, void *);
					sigset_t   sa_mask;//函数指向期间的mask值
					int        sa_flags;//默认为0，表示动作执行期间本信号忽略
					void     (*sa_restorer)(void);
				};
```



### 共享内存

#### mmap
- void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
- int munmap(void *addr, size_t length);//解除共享内存
参数：
	- addr：传出参数，指向共享内存地址
	- length：共享内存长度（必须是1024整数倍，是整页）
	- prot：共享内存的读写权限
				PROT_EXEC //页内容可以被执行
				PROT_READ //页内容可以被读取（常用）
				PROT_WRITE //页可以被写入（常用）
				PROT_NONE //页不可访问

	- flags：共享内存模式
				  MAP_ANONYMOUS //匿名映射
				  MAP_SHARED //与其它所有映射这个对象的进程共享映射空间（常用）
					
	- fd：共享文件的描述符
	- offset：从文件fd中的开始位置
```
int main(void)
{
	char *mem;

//	int fd = open("hello244", O_RDONLY|O_CREAT|O_TRUNC, 0644);
	int fd = open("dict.txt", O_RDWR);
	if (fd < 0)
		sys_err("open error");
/*
	   len = lseek(fd, 3, SEEK_SET);   //获取文件大小,根据文件大小创建映射区
	   write(fd, "e", 1);              //实质性完成文件拓展
	   printf("The length of file = %d\n", len);
*/
	mem = mmap(NULL, 1024, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
	if (mem == MAP_FAILED)            //出错判断
		sys_err("mmap err: ");

	close(fd);

	strcpy(mem, "xxx");
	printf("%s\n", mem);

	if (munmap(mem,  1024) < 0)
		sys_err("munmap");

	return 0;
}
```

#### shmat

- shmget获取（创建）一块共享内存
	- int shmget(key_t key, size_t size, int shmflg) 返回shmid
	- key：内存编号
	- size：内存大小
	- shmflg：内存权限eg:0660|IPC_CREAT若不存在创建

- shmat链接共享内存
	- void *shmat(int shmid, const void *shmaddr, int shmflg)
	- shmid：共享内存id
	- shmaddr：指定共享内存地址（一般传NULL,由内核指定）
	- shmflg：一般传0，表示可读可写

- 	shmdt(p);取消连接共享内存		
	- int shmdt(const void *shmaddr)
- shmctl删除共享内存
   -  shmctl(int shmid, int cmd, struct shmid_ds *buf);
   -  cmd：IPC_RMID删除这片共享内存
   -  buf共享内存管理结构体。一般传空
```cpp
int main()
{
	int shmid = 0;
	//创建共享内存
	//相当于打开共享内存( 打开文件)
	//若文件存在  则使用旧的 
	//若文件不存在 则创建
	shmid = shmget(0x1111, 100, 0666 |IPC_CREAT );
	if (shmid < 0)
	{
		printf("func shmget() err\n");
		return 0;
	}
	printf("创建共享内存 ok\n");
	
	
	//连接共享内存
	//	void *shmat(int shmid, const void *shmaddr, int shmflg);
	
    void *p = shmat(shmid, NULL, 0);
    printf("%d\n",p);
    
    strcpy((char *)p, "111122234344");   
    printf("p:%s \n", p);    
    printf("任意键删除连共享内存\n");
    getchar();
	//取消连接共享内存
	shmdt(p);
	//删除共享内存
    shmctl(shmid, IPC_RMID, NULL);
    
	return 0;
}

```


### 本地套接字

 socket(AF_UNIX, SOCK_STREAM, 0)
