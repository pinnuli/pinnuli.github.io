---
title: git分支管理常用命令
date: 2018-03-11 15:26:34
categories: "git" 
tags:
	- git
---

创建分支：


	git branch branch_name


切换分支：

	git checkout branch_name
创建并切换分支
	
	git checkout -b branch_name


创建远程分支到本地：

	git checkout -b branch_name origin/branch_name	


查看当前分支：


	git branch

获取所有分支：

	git fetch

合并某分支到当前分支：

	git merge branch_name
禁用Fast forward（快速合并）， 普通模式合并：

	git merge --no-ff -m "merge with no-ff" branch_name

(这里会在合并的时候自动生成一个新的commit)

删除分支：

	git branch -d branch_name

强制删除分支（用于为合并就删除时）：

	git branch -D branch_name


查看分支合并图：

	git log --graph

保存分支工作现场：

	git stash

查看保存列表：

	git stash list


恢复保存状态：

	git stash apply
	git stash apply stash@{x}

删除保存状态：

	git stash drop 
	git stash drop stash@{x}

恢复并删除保存状态：

	git stash pop

推送分支到远程仓库：

	git push origin branch_name

建立本地分支与远程分支的关联：
	
	git branch --set-upstream branch-name origin/branch-name


