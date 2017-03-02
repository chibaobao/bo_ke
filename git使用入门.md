---
title: git使用入门
date: 2016-10-09 18:44:46
tags: github
categories: github相关
---

# github 使用

## githhub中的名词

- Issue:(任务问题)创建pull Request 时会被创建，
- Wiki:开发文档或手册
- Pull Request:合并请求的仓库

## git仓库本地使用
1.  init  初始化本地仓库
2. 编写文件
	- 如：vim a.c 
3. 提交和添加

	3.1. 添加到缓存区 add a.c
	3.2.  提交 commit
	-  commit a.c                            提交a.c
	-  commit -m  "aaa"                  提交缓存区中全部内容，并添加信息aaa

	3.1and2.  
	 commit -am "abc"             提交且添加相当于3.1加3.2

## 本地仓库连接远程仓库

git push -u origin(标识符) master(分支名)  将master推送到origin
1. 配置好ssh
2. git remote add origin(标识符) git@.......  添加远程仓库
3. git push -u origin(标识符) master(分支名)  将master推送到origin

***注意***：同步远程仓库正确做法

1. pull
2. 改代码
3. pull
4. commit
5. push

## git常用命令

```
git init : 初始化当前目录为git仓库
git status 查看git仓库状态
git add filename-->git commit filename 提交到缓存区-->保存到仓库
git commit -m "abc"  记述提交信息(不是第一次提交可直接用该命令)
git commit -am "abc"  合并了add和commit -a两个命令
git log  :查看(以当前状态为终点的)日志文件
git log --pretty=short :只查看提交的第一条信息
git reflog 查看当前仓库执行过的操作日志
git log filename   :只查看filename文件记录
git diff  :查看工作树与暂存区的区别
git diff HEAD :查看工作树与最新一次提交的区别
git branch  :查看分支
git branch -a 同时查看当前分支和远程分支
git checkout -b name  :创建名为name的分支并进入
=git branch name  +  git  checkout name
git checkout -  切换到上一个分支
git merge --no-ff name  将名为name的分支合并的主干，并记录
git reset --hard 哈希值  ：回溯到该哈希值的位置
git commit --amend   :修改提交的commit内容
git rebase -i HEAD~2 将最后两次HEAD值的修改合并(合并小错误) 
git remote origin(标识符) git@.......  添加远程仓库
git push -u origin(标识符) master(分支名)  将master推送到origin
git clone git@....  下载远程仓库到本地目录(注意本地目录不能已经存在仓库).
git clone -b 分支名 git@....  下载远程仓库到本地目录(指定分支)
git push <-->git pull 将当前分支同步到远程仓库，从远程仓库同步到本地仓库
```

## git冲突解决

如果git pull  如果本地代码 和远程代码出现冲突，需要手动解决。

1. git stash  //将本地文件 回馈 上一步操作

2. 在上一个版本下，执行git pull 将新代码下载下来

3. 回复最后写的代码 git stash pop

4. 解决冲突部分：查看冲突文件，里面会有 <<<<<  和 >>>>>这里之中就是本地和远程的冲突的部分，手动去解决

5. 然后再 git pull, git commit , git push 
