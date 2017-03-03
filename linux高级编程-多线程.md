---
title: linux高级编程-多线程
date: 2016-11-01 23:35:08
tags: Linuxc
categories: 编程学习
---

# 多线程

- 多线程编译时加 -pthread
- pthread_t  == usinend long
- ps -Lf pid  //查看线程

## 多线程创建

- 多线程创建
	- int pthread_create(*thread,NULL（线程属性）,执行主逻辑回调函数，回调函数参数arg);  
	- 回调函数eg: void *fun(void *arg)	

- 线程回收（detach和join只能选一个否则会出错）
	- pthread_detach(threadid)
		- 设置线程为游离态
		- threadid要设置的线程id
	- pthread_join(thread,void **retval);
		- thread线程号
		- retval用来存储线程返回信息

	```cpp
	其他函数
	pthread_t pthread_self(void);//pthread_self是一种函数，功能是获得线程自身的ID。
	char* streeror(errno);查看错误号信息
	pthread_cancel(thread);//kill 子线程，子线程中要有到达取消点
		pthread_testcncel();检查是否到的检查点
	pthread_exit();线程退出
	pthread_equal(t1,t2);//判断t1,t2是不是同一个线程
	```


```
#include<iostream>
#include <pthread.h>
#include <unistd.h>
class A
{
public:
        int m_a;
        A() 
        {   
                std::cout<<"creat A"<<std::endl;
        }   
        ~A()
        {   
                std::cout<<"delete A"<<std::endl;
        }   
};
A * b = new A;
void *call_back(void *arg)
{
        sleep(1);
        A * a = (A*)arg;
        std::cout<< a->m_a<<std::endl;
        std::cout<< b->m_a<<std::endl;
}
void *call_back2(void *arg)
{
        sleep(1);
        int *p=(int *)arg;
        std::cout<<"c:"<<*p<<std::endl;
}
int main()
{
        A * a = new A;
        a->m_a = 1;
        b->m_a = 1;
        int c=1;
        int pid;
        pthread_t pthread;
        pthread_create(&pthread, NULL, call_back, a); 
        pthread_detach(pthread);//设置线程分离（自动回收线程）
        pthread_create(&pthread, NULL, call_back2, &c);
        pthread_detach(pthread);//设置线程分离（自动回收线程）
        a->m_a = 10;
        b->m_a = 10;
        c=10;
        std::cout<< a->m_a<<std::endl;
        std::cout<< b->m_a<<std::endl;
        std::cout<<"#########################"<<std::endl;
        //      delete a;
        //      delete b;
        sleep(2);//若主函数退出，各个线程将全部结束
        return 0;
}
```

结果：
```
creat A
creat A
10
10
#########################
c:10
10
10
```

## 多线程锁

- 互斥量：
	- 类型：pthread_mutex_t
	- pthread_mutex_init
	- pthread_mutex_init(&mutex,NULL);
	- pthread_mutex_lock(mutex);
	- pthread_mutex_unlock(mutex);
	- pthread_mutex_trylock(mutex);
	
- 读写锁(写独占，读共享，写优先级高)：
	- 读锁：读时共享，写时阻塞
	- 写锁：读或写都阻塞
	- 多锁：之后写锁，读锁，读也阻塞	
	- 类型：pthread_rwlock_t
	- pthread_rwlock_init()
	- pthread_rwlock_destroy()
	- pthread_rwlock_lock()
	- pthread_rwlock_unlock()
	- pthread_rwlock_trylock()

- 条件变量：
	- 类型:pthread_cond_t
	- pthread_cond_init()
	- pthread_cond_destroy()
	- pthread_cond_wait(cond,mutex)//条件不满足解锁，条件满足，(如果解锁了)尝试再次加锁
	- pthread_cond_timewait(cond,mutex)
	- pthread_cond_signal()//条件满足，唤醒至少一个wait
	- pthread_cond_broadcast()//条件满足，唤醒全部wait

- 信号量
	- sem_init(&sem,线程 0 or进程 1,信号量初始值,)
	- set_destroy(&sem)
	- sem_wait()      ==lock
	- sem_post()		==unlock

- 文件锁应用于进程间
	- fcntl:
		- F_SETLKW	lock/wait
		- F_SETLK		trylock
		- F_GETLK		
		- struct flock:
			l_type F_RDLCK,F_WRLCK.F_UNLCK
	  l_whence SEEK_SET SEEK_.....
