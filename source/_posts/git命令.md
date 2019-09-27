---
title: git命令
date: 2019-09-27 14:36:33
tags:
---

git init    初始化

git status  查看仓库当前的状态

git diff <file>  查看具体修改了什么内容

git diff HEAD --<file> 命令可以查看工作区和版本库里面最新版本的区别

git add <file> 添加到暂存

git add -f <file>  假如文件被忽略这样可以强制添加

git commit -m "balabalabala" 提交暂存区的文件到本地仓库

git log --graph --pretty=oneline 查看日志

git reset --hard HEAD^   (git reset --hard 版本编号)  版本回退 

git reset HEAD <file> 添加到了暂存区时，想丢弃修改 

git checkout --<file>  当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时

git rm <file> 从版本库中删除该文件(然后commit)

git remote   查看远程库信息

git remote -v  更加详细的查看

git remote add origin 地址  本地关联远程库

git clone  地址   克隆远程库

git branch  查看当前分支 

git branch <name>   创建分支

git checkout <name>   切换分支

git checkout -b <name>  我们创建分支，然后切换到分支

git merge <name>  合并分支到当前分支

git merge --no-ff -m "xxxxx" <name>   合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并

git branch -d <name>   删除分支

git branch -D <name>   强行删除

git pull origin <name>   拉取

git push origin <name>  推送

git stash    把当前工作现场“储藏”起来，等以后恢复现场后继续工作

git stash list   查看贮藏区

git stash apply  恢复后，stash内容并不删除

git stash drop    删除贮藏区的内容

git stash pop     恢复的同时把stash内容也删了

你可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令 git stash apply stash@{0}

git rebase  变基(线路变得好看)

git tag <tagname>   打标签

git tag -a <tagname> -m "balabalbal..."   可以指定标签信息

git tag   查看所有标签

git show <tagname>   查看该标签版本信息

git tag -d <tagname>  删除标签

git push origin <tagname>  推送标签到远程

git push origin --tags   一次性推送全部尚未推送到远程的本地标签

git push origin :refs/tags/<tagname>  可以删除一个远程标签

在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。

git reflog用来记录你的每一次命令(强力后悔药!!!!!!!!!!)
