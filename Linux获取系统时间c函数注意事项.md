
---
title: Linux获取系统时间c函数注意事项
date: 2016-11-06 20:45:46
tags: Linuxc
categories: 编程学习
---

# Linux获取系统时间c函数注意事项

## 获取当前时间
获得当前系统时间
1. time(&now);
	- 获取当前时间（是一个长整形时间戳）
2. timenow = localtime(&now);
	-  转化为一个结构体，结构体成员包括十分月日年
3. asctime_r(timenow,strtemp);或asctime(timenow);
	- 将结构体转化为字符串


```cpp
#include <time.h>
#include <stdio.h>
int main()
{
    time_t now;   
    struct tm *timenow;   
    char strtemp[255];   
    char *str_time;

	time(&now);   
    printf("time is : %ld \n", now);  
    timenow = localtime(&now);   
    str_time = asctime(timenow);//不安全版本
    asctime_r(timenow,strtemp);//安全版
```

## asctime_r和asctime注意事项，对比
asctime_r和asctime_r对比，会感觉asctime有内存泄漏的可能，其实并不会内存泄漏但有一定不可控性
asctime返回的地址是一块静态地址，等下次调用asctime返回的还是这块地址，因此并不会内存泄漏，但是有一定不可控性，比如使用了分别获取了两次当前世界函数，然后再读取就会发现两次时间相同，eg:

```cpp
#include <time.h>
#include <stdio.h>
int main()
{
    time_t now;   
    struct tm *timenow;   
    char strtemp[255];   
    char *str_time;
    char *str_time2;
    
    //第一次获取当前时间
    time(&now);   
    printf("time is : %ld \n", now);  
    timenow = localtime(&now);   
    str_time = asctime(timenow);//不安全版本
    printf("local time is : %s \n", str_time);  
    sleep(1);
    //第二次获取当前时间
    time(&now);   
    timenow = localtime(&now);   
    str_time2 = asctime(timenow);//不安全版本
    //打印str_time1 和str_time2 发现结果相同
    printf("local time1 is : %s \n", str_time);  
    printf("local time2 is : %s \n", str_time2);  
    //free(str_time);//如果释放会出错
    asctime_r(timenow,strtemp);
    printf("local time is : %s \n", strtemp);  

}
```