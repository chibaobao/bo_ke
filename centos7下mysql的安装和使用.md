---
title: centos7下mysql的安装和使用
date: 2016-11-09 00:40:01
tags: mysql
categories: 编程学习
---

# centos7 mysql 安装和使用

centos7中的mysql已经被mariadb取代，这里时间安装的是mariadb

## 安装--初始化--登录

```
//安装
root@server0 ~]# yum -y groupinstall mariadb mariadb-clients

//设置systemctl开机启动
[root@server0 ~]# systemctl enable mariadb
[root@server0 ~]# systemctl restart mariadb.service 

//添加防火墙
[root@server0 ~]# firewall-cmd --permanent --add-service=mysql
[root@server0 ~]# firewall-cmd --reload 

//初始化基本设置
[root@server0 ~]# mysql_secure_installation 			//提升安全
Enter current password for root (enter for none): 		//没有密码，直接回车

Set root password? [Y/n] 

Remove anonymous users? [Y/n] 

Disallow root login remotely? [Y/n] 

Remove test database and access to it? [Y/n] 

Reload privilege tables now? [Y/n] 


//登录
[root@server0 ~]# mysql -uroot -p
```

## 安装c的API库
- find / -name "mysql.h"
	- 如果找不到就执行以下命令安装
- yum -y install  mysql-devel

## 使用

### 创建数据库（中文乱码问题）
- 创建一个名称为mydb1的数据库。
	- create database mydb1;

- 创建一个使用utf-8字符集的mydb2数据库。
	- create database mydb2 character set utf8;

- 创建一个使用utf-8字符集，并带校对规则的mydb3数据库。会对存入的数据进行检查。
	- create database mydb3 character set utf8 collate utf8_general_ci;

### 查看数据库

- 显示所有数据库
	- how databases;
- 显示创建数据库的语句信息
	- show create database mydb2; 

### 修改数据库（utf8）

alter database mydb1 character set utf8;	

## 中文乱码问题解决

三层因素：
- 因素1： MySQL自身的设计 

	- 【实验步骤1】：	
mysql> show variables like 'character%';	 查看所有应用的字符集

	- 【实验步骤2】：	
		$ mysql -uroot -p123456 --default_character_set=gbk 指定字符集登录数据库
			mysql> show variables like 'character%';
			影响了与客户端相关联的 3处 (最外层)
		在这种状态下执行use mydb2;
			mysql> select * from employee;			
			查看输出，会出现乱码。
			原来的三条数据，是以utf8的形式存储到数据库中，当使用gbk连接以后，数据库仍按照utf8的形式将数据返回，出错。
	- 【实验步骤3】：
		在该环境下插入带有中文的一行数据。
			mysql> insert into employee(id,name,sex,birthday,salary,entry_date,resume) values(10,'张三疯',1,'1983-09-21',15000,'2012-06-24','一个老牛');
			ERROR 1366 (HY000): Incorrect string value: '\x80\xE4\xB8\xAA\xE8\x80...' for column 'resume' at row 1

- 因素2：操作系统的语言集
	linux操作系统 是一个 多用户的操作
	[root@localhost ~]# cat /etc/sysconfig/i18n
	LANG="zh_CN.UTF-8"	
	操作系统的菜单按照zh_CN显示,  文件存储按照utf8
	linux操作系统语言环境 和 用户的配置的语言环境LANG 相互影响
	[mysql01@localhost ~]$ echo $LANG
	zh_CN.UTF-8
	
	- 【实验步骤4】： 
		修改用户下的.bash_profile 中的LANG，屏蔽操作系统的LANG设置。再查数据库
				mysql> select * from employee;
		结论： 用户的LANG设置，影响了应用程序的语言环境,导致myql的语言环境发生了改变：
			mysql> show variables like 'character%';
			在此环境下，检索中文会出现乱码。
	-  【实验步骤5】：在上述环境之下，向数据库中插入中文。
		insert into employee(id,name,sex,birthday,salary,entry_date,resume) values(5,'张三疯',1,'1987-05-21',15000,'2014-06-24','一个老牛');
		数据能插入到数据库中，没 有 报 任 何 错 误！但显示不正确。

- 因素3：文件存储格式 

