---
title: makefile_and_gcc
date: 2016-10-23 22:53:54
tags: 
    - gcc
    - makefile
categories: 编程学习
---
# makefile和gcc

## makefile

- 注意:makefile中目标:源  顶行首写，不用缩进
- $make clean -n 测试clean执行结果，但不执行

```makefile
src = $(wildcard *.c)     #查找当前文件全部.c文件
obj2 =$(patsubst %.c,%.o,$(src))	#src-->%.c--->%.o       将$src以%.c匹配，转换为%.o  如a.c匹配为a.o
obj =$(patsubst %.o,%,$(obj2) )   

All:$(obj)   
$(obj):%:%.o			#$(obj)静态规则,可以理解为目标是从$(obj)中匹配，目标%，源%.c
	gcc $< -o $@		#  '$<'  依次取,  '@'表示%所表示的参数
$(obj2)：%.o:%.c		
	gcc -c $< -o $@		

clean:
	-rm -rf $(obj) $(obj2) 	#删除$(obj) $(obj2)，-rf如果出错跳过
.PHONY: clean ALL			#表示clean ALL 是伪目标不会形成相应名称的文件

```
a eg for a test makefile:
```makefile
obj=a b c.o 
obj2=a b
obj3=c.o
all:$(obj) 
$(obj2):%:%.c
    gcc $< -o $@
$(obj3):%.o:%.c
    gcc $< -o $@
#%:%.c
#   gcc $< -o $@
clean:
      -rm -rf $(obj) $(ibj2)
.PHONY: all clean
```

## gcc

```
-E预处理
-S汇编
-c编译
-I头文件目录
-Wall给更多警告
-L动态库目录
-l动态库名
-o输出文件名
```

## 动态库
查看动态库中的信息 
- nm **.so
- readelf -a **.so

