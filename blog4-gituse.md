---
title: git学习和日常使用
date: 2021-10-23 22:00:00
tags: 日常学习
cover: /img/cloud.jpg
categories: 工具
description: git学习总结,以后用到git直接在这里查看
sticky: 1
---

## git 原理简单解读

一个版本管理工具,用 c 写的.主要分为三部分:
工作区(操作的本地文件夹内)
暂存区(git add 加入的地方,和本地区与版本库交互)
版本库(commit 之后放到的地方)

## git 常用操作

### 下载远程库

`git clone`(这个就不用多说了)

### 提交本地工作区文件到 github 远程库

1. 本地创建完成后,`git add .`把所有文件加入到暂存区
2. `git commit -m ""` 把暂存区文件推送到本地分支
3. 创建远程库
4. `git pull origin master` 如果报错参考报错 1
5. `git push (-u) origin master`

### 版本回退

版本回退一定是 commit 之后才会发生的,如果都没有提交就谈不上版本

- `git log`可以查看历史 commit,并且含有很重要的 id 需要回退时使用;`git reflog`可以查看命令历史
- 回退上一个版本:`git reset --hard HEAD^`
- 回退指定版本: `git reset --hard (id)`id 内容就是刚刚说的 id,输入前几位就行

### 撤销修改

修改一般是在暂存区的

- `git checkout --文件名`丢弃工作区的修改

### 删除和恢复

从工作区删除文件之后,版本库还没有删除,这时 git status 就会提醒你有不同的地方

- 如果确定删除:`git rm test.txt`
- 如果想恢复: `git checkout -- test.txt`其实是用版本库里的版本替换工作区的版本

### 分支操作

- 创建分支:`git branch dev`
- 查看分支:`git branch`
- 切换分支：`git checkout <name>`或者`git switch <name>`
- 创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`
- 合并某分支到当前分支：`git merge <name>`
- 删除分支：`git branch -d <name>`

### 标签

- `git tag v1.0`可以给分支打一个标签,`git tag`可以查看所有标签
- `git show v1.0`可以查看当前版本信息,修改/作者/提交日期等等

### pull request

如果多人合作一个 github 库,更改后不能直接提交到 master,需要先创建自己的分支,然后发起 pull request.
具体操作是:

1. 首先在本地 add commit 然后创建一个不是 main/master 的新分支
2. git pull 把远程的**主分支**pull 下来到你的新分支去。
3. 用 git status 查看一下是不是还有没合并的
4. 合并成功,push 上去.**记得一定是新的分支,不要 push 到主分支**
5. 在 github 上发起 pull request 即可

### rebase

在 pull 的时候,如果出现提交失败可以考虑
暂时还没有遇到这种情况,所以先放个链接
https://www.liaoxuefeng.com/wiki/896043488029600/1216289527823648

## git 常用命令

- git clone
- git add
- git status 查看仓库当前状态并显示有变更的文件
- git diff 比较文件的不同，即暂存区和工作区的差异。
- git commit 提交暂存区到本地仓库。
- git log 查看历史提交记录
- git remote 远程仓库操作
  - `git remote -v` 查看远程仓库
  - `git remote add origin ...` 添加远程仓库
  - `git remote rm origin` 删除远程仓库
- git reset 回退版本
  - `git reset HEAD^` 回到上一个版本 回退几个版本就是几个^,多的话可以 HEAD~3
  - `git reset --soft(--head) HEAD^` --soft 命令回退到某个版本;--hard 撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交：

## git 报错总结

### 报错 1:

Updates were rejected because the remote contains work that you do not have locally
解释:本地库和远程库有不一样的
解决:需要先 pull 一下

1. `git pull origin master --allow-unrelated-histories`
2. (如果出现一个输入界面:Please enter a commit to explain why this merge is necessary?(merge branch))按 ESC,然后输入`:wq`即可
3. `git push (-u) origin master` (如果不是第一次就不要-u)

## git 和 vscode

详情可以看这里,讲的很详细:https://zhuanlan.zhihu.com/p/23344403
