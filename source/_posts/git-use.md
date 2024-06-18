---
title: Git使用教程
date: 2023-03-12 18:28:40
categories: 前端
tags:
 - Git
 - 教程
---

## 一、常用命令：
* mkdir XX：创建一个空目录 XX指目录名
* pwd：显示当前目录的路径
* cat xx：查看xx文件内容
* <font color="#dd0000"> git init：把当前的目录变成可以管理的git仓库，生成隐藏的.git文件夹
* git add xx：把xx文件添加到暂存区
* git commit -m “xx”：提交文件 -m后面的是注释，必须写！</font>
* git status：查看仓库状态
* git log：查看历史记录
* git reset --hard HEAD^：往上回退一个版本
* git reflog：查看历史记录的版本号id
* git checkout -- xx：把xx文件在工作区的修改全部撤销
* git rm xx：删除xx文件
* <font color="#dd0000">git remote add origin https://github.com/xxxxx/a.git 关联一个远程库
* git push -u（第一次尽量加上-u，以后不用）origin master：把当前master分支推送到远程库
* git clone https://github.com/xxxxx 从远程库中克隆
* git checkout -b dev：创建dev分支 并切换到dev分支上
* git branch：查看当前所有的分支
* git checkout master：切换回master分支
* git merge dev：在当前分支合并dev分支</font>
* git branch -d dev：删除dev分支
* <font color="#dd0000">git branch xxx：创建分支xxx</font>
* git remote：查看远程库信息
* git remote -v查看远程库的详细信息
* <font color="#dd0000">git pull origin master 将远程库的更新拉取到本地并自动合并
* git push origin master：git会把master分支推送到远程库对应的分支上</font>

----

## 二、GitHub远程仓库的使用

### 场景一（关联）：本地有仓库，要和远程仓库做关联
右键git bash here，输入命令
```
git init
git add .
git commit -m “first commit”
```
在GitHub上创建一个远程仓库
```
git remote add origin https://github.com/xxx/xxx.git (HTTPS) #仓库地址根据实际选择
```
备注：如果此步关联错了，解决办法如下。
* 暴力解决：删除.git文件夹，重新建立本地仓库。
* 优雅解决：git remote remove origin，再在重新关联仓库

### 场景二（推送）：本地有仓库有内容，要推送给远程库
```
git push -u origin master （首次加-u）
```
根据提示输入用户名密码
我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送到远程新的master分支，还会把本地master分支和远程的master分支关联起来，在以后的推送时可以简化命令`git push origin master`。
备注：正常情况下，成功推送一次后，电脑会记住和账号与密码，下次推送时不会再提示输入。若在电脑不能够自动记住github的账户和密码，需执行以下命令解决：`git config --global credential.helper store`。

### 场景三（拉取）：本地有仓库有内容，获取远程库新内容
+ **第一种拉取方式：** `git pull origin master`，
将远程仓库的master分支上代码版本复制/合并到本地master分支上
+ **第二种拉取方式：**`git fetch origin master:tmp`
新建一个tmp分支，将远程仓库的master分支上代码版本复制到tmp分支上，不会自动合并。

### 场景四（克隆）：本地无仓库，要获取一个完整的远程库
备注：只在第一次获取远程库时才需要克隆
```
git clone https://github.com/xxx.git (HTTPS)
```