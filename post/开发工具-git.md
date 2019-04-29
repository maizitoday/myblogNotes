---
title:       "git常用命令"
subtitle:    ""
description: ""
date:        2019-04-28
author:      "麦子"
image:       ""
tags:        ["开发工具", "git"]
categories:  ["Tech" ]
---



# Git简单说明

保留原文链接：https://www.yiibai.com/git/getting-started-git-basics.html

Git 有三种状态，你的文件可能处于其中之一：已提交(committed)、已修改(modified)和已暂存(staged)。 已提交表示数据已经安全的保存在本地数据库中。 已修改表示修改了文件，但还没保存到数据库中。 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：**Git 仓库、工作目录以及暂存区域**。工作目录、暂存区域以及 Git 仓库如下图所示 -

![img](http://www.yiibai.com/uploads/images/201707/0607/744160702_48164.png)

Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。

工作目录是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。 有时候也被称作‘索引’，不过一般说法还是叫暂存区域。

基本的 Git 工作流程如下：

1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

如果 Git 目录中保存着的特定版本文件，就属于已提交状态。 如果作了修改并已放入暂存区域，就属于已暂存状态。 如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。 在Git 基础一章，你会进一步了解这些状态的细节，并学会如何根据文件状态实施后续操作，以及怎样跳过暂存直接提交。

# 常用命令

**注意： git只会对文件有反应，如果创建一个空文件夹，git status是不会有反应的。**



## help

查看某一个命令的帮助

```properties
git help clone 
#按 F 可以向下查看帮助
#按 B 可以向上查看帮助
#按 q 退出帮助
```



## config

Git 前的配置，设置你的全局的用户，邮箱，打开的编辑器，对比工具，颜色等。 如果是具体到某一个项目也可以在进行设置。

```properties
#这个文件也同时在你当前用户的
cat ~/.gitconfig
 
#查看你的git配置
git config --global --list
 
#查看Git认为的一个特定的关键字目前的值
git config --global user.name
 
#增
git config  --global --add user.name jianan
#全局设置你的用户名和邮箱
$ git config --global user.name "wirelessqa"  
$ git config --global user.email wirelessqa.me@gmail.com  

#删
git config  --global --unset user.name
 
#改
git config --global user.name jianan
```



## init

初始化一个项目，如果不需要这个项目了，就把这个.git删除掉。 这个我们一般不需要改动。

```properties
➜  Desktop mkdir gitDemo
➜  gitDemo git init
Initialized empty Git repository in /Users/maizi/Desktop/gitDemo/.git/
➜  gitDemo git:(master) ls -a
.    ..   .git
➜  gitDemo git:(master)
➜  .git git:(master) ll
total 24
-rw-r--r--   1 maizi  staff    23B Apr 29 17:33 HEAD
#这个config是项目的配置文件
-rw-r--r--   1 maizi  staff   137B Apr 29 17:33 config 
-rw-r--r--   1 maizi  staff    73B Apr 29 17:33 description
drwxr-xr-x  13 maizi  staff   442B Apr 29 17:33 hooks
drwxr-xr-x   3 maizi  staff   102B Apr 29 17:33 info
drwxr-xr-x   4 maizi  staff   136B Apr 29 17:33 objects
drwxr-xr-x   4 maizi  staff   136B Apr 29 17:33 refs
```



## commit

一次将暂存区中的内容提交到版本库中， 通过git status状态可以看出详细的流程。

```properties
➜  gitDemo git:(master) git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
➜  gitDemo git:(master) touch maizi.text
➜  gitDemo git:(master) ✗ ls
maizi.text
➜  gitDemo git:(master) ✗ vi maizi.text
➜  gitDemo git:(master) ✗ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	maizi.text

nothing added to commit but untracked files present (use "git add" to track)
➜  gitDemo git:(master) ✗ git add maizi.text
➜  gitDemo git:(master) ✗ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   maizi.text

➜  gitDemo git:(master) ✗ git commit -m '提交说明信息'
[master (root-commit) 2391282] 提交说明信息
 1 file changed, 2 insertions(+)
 create mode 100644 maizi.text
➜  gitDemo git:(master) git status
On branch master
nothing to commit, working tree clean

#查看提交的信息
➜  gitDemo git:(master) git log
commit 2391282c01732cf6c3269fafc62bccefa55f1dae (HEAD -> master)
Author: maizitoday <maizi_today@sina.com>
Date:   Mon Apr 29 17:39:13 2019 +0800

    提交说明信息
(END)
```



## diff

工作区(work dict)和暂存区(stage)的比较，为了防止工作区的修改没有被提交到暂存区中，或则查看工作区和暂存区之间的差异，可以使用git diff指令。

### 缓存区比较

```properties
➜  gitDemo git:(master) ✗ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   maizi.text

no changes added to commit (use "git add" and/or "git commit -a")
➜  gitDemo git:(master) ✗ git diff maizi.text

#同时对比完成后， 需要add后，然后才能commit，  如果直接commit，文件是无法提交的。
```



### 分支比较

```properties
➜  gitDemo git:(company) git diff master..company

#具体比较两个分支里面的一个文件
➜  xiaoqiang git:(company) ls
tody.txt
➜  xiaoqiang git:(company) git diff  master..company tody.txt
```





## rename

重命名已经命名的文件，下面的mv命令更加简单。

```properties
➜  gitDemo git:(master) ls
maizi.text
➜  gitDemo git:(master) mv maizi.text maizi_today.text
➜  gitDemo git:(master) ✗ ls
maizi_today.text
➜  gitDemo git:(master) ✗ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    maizi.text

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	maizi_today.text

no changes added to commit (use "git add" and/or "git commit -a")
➜  gitDemo git:(master) ✗ git rm maizi.text
rm 'maizi.text'
➜  gitDemo git:(master) ✗ git add maizi_today.text
➜  gitDemo git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	renamed:    maizi.text -> maizi_today.text

➜  gitDemo git:(master) ✗ git commit -m  "重命名"
[master 3f4add1] 重命名
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename maizi.text => maizi_today.text (100%)
➜  gitDemo git:(master)
```



## mv

重命名和移动文件， 和Linux一样。

```properties
#重命名 
➜  gitDemo git:(master) git mv maizi_today.text tody.txt
➜  gitDemo git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	renamed:    maizi_today.text -> tody.txt

➜  gitDemo git:(master) ✗ git commit -m "重命名文件"
[master 5491bd8] 重命名文件
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename maizi_today.text => tody.txt (100%)
➜  gitDemo git:(master) git status
On branch master
nothing to commit, working tree clean
➜  gitDemo git:(master)

#移动文件
➜  gitDemo git:(master) ls
tody.txt  xiaoqiang
➜  gitDemo git:(master) git mv tody.txt xiaoqiang
➜  gitDemo git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	renamed:    tody.txt -> xiaoqiang/tody.txt

➜  gitDemo git:(master) ✗ git commit -m "移动目录"
[master 4b048a4] 移动目录
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename tody.txt => xiaoqiang/tody.txt (100%)
➜  gitDemo git:(master) ls
xiaoqiang
➜  gitDemo git:(master)
```



## rm

```properties
➜  gitDemo git:(master) git rm -r xiaoqiang
rm 'xiaoqiang/tody.txt'
➜  gitDemo git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	deleted:    xiaoqiang/tody.txt

➜  gitDemo git:(master) ✗ git commit -m "删除文件"
[master fcf6d55] 删除文件
 1 file changed, 4 deletions(-)
 delete mode 100644 xiaoqiang/tody.txt
➜  gitDemo git:(master) git status
On branch master
nothing to commit, working tree clean
➜  gitDemo git:(master)
```



## checkout

命令用于切换分支或恢复工作树文件

### 切换分支

```properties
➜  gitDemo git:(work) git checkout company #切换分支
```

### 

### 恢复工作树文件

```properties
#删除文件，还没有提交的文件恢复
➜  gitDemo git:(master) git rm xiaoqiang/tody.txt
rm 'xiaoqiang/tody.txt'
➜  gitDemo git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	deleted:    xiaoqiang/tody.txt

➜  gitDemo git:(master) ✗ ls
#   HEAD ： 表示最近一次提交     --：表示当前分支
➜  gitDemo git:(master) ✗ git checkout HEAD -- xiaoqiang/tody.txt
➜  gitDemo git:(master) git status
On branch master
nothing to commit, working tree clean
➜  gitDemo git:(master) ls
xiaoqiang
➜  gitDemo git:(master) cd xiaoqiang
➜  xiaoqiang git:(master) ls
tody.txt
➜  xiaoqiang git:(master)

#---------------------------------------------#

#删除文件，已经提交了， 进行文件恢复
➜  gitDemo git:(master) ls
xiaoqiang
➜  gitDemo git:(master) git rm -rf xiaoqiang
rm 'xiaoqiang/tody.txt'
➜  gitDemo git:(master) ✗ git commit -m "删除文件"
[master b25edc3] 删除文件
 1 file changed, 4 deletions(-)
 delete mode 100644 xiaoqiang/tody.txt
➜  gitDemo git:(master) ls
#  HEAD^: 表示最近的提交的上一次提交。 HEAD^^ 上两次的提交  --：表示当前分支
➜  gitDemo git:(master) git checkout HEAD^ -- xiaoqiang
➜  gitDemo git:(master) ✗ ls
xiaoqiang
➜  gitDemo git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   xiaoqiang/tody.txt

➜  gitDemo git:(master) ✗ git commit -m "恢复删除文件"
[master ce2420c] 恢复删除文件
 1 file changed, 4 insertions(+)
 create mode 100644 xiaoqiang/tody.txt
➜  gitDemo git:(master) ls
xiaoqiang
➜  gitDemo git:(master)
```



## revert

恢复到某一个提交之前的数据。

```properties
gitDemo git:(master) git log --oneline
ce2420c (HEAD -> master) 恢复删除文件
b25edc3 删除文件
c0c668d 恢复已经删除文件
fcf6d55 删除文件
4b048a4 移动目录
5491bd8 重命名文件
3f4add1 重命名
e2fc445 测试
2391282 提交说明信息
(END)
gitDemo git:(master) git revert ce2420c
#数据恢复到这个log记录之前
```



## reset

感觉就是设置一个指针， 这个指针会告诉我们下次或者这次提交代码时候，重新的一个起点指针。

```properties
➜  gitDemo git:(master) git log --oneline
dcb0636 (HEAD -> master) Revert "删除文件"
341fa18 删除文件
7a39e4a Revert "测试"
ce2420c 恢复删除文件
b25edc3 删除文件
c0c668d 恢复已经删除文件
fcf6d55 删除文件
4b048a4 移动目录
5491bd8 重命名文件
3f4add1 重命名
e2fc445 测试
2391282 提交说明信息


git reset (–mixed) HEAD~1 | git reset (–mixed)  341fa18
回退一个版本,且会将暂存区的内容和本地已提交的内容全部恢复到未暂存的状态,不影响原来本地文件(未提交的也 
不受影响) 

git reset –soft HEAD~1 | git reset –soft 341fa18
回退一个版本,不清空暂存区,将已提交的内容恢复到暂存区,不影响原来本地的文件(未提交的也不受影响) 

git reset –hard HEAD~1 | git reset –hard 341fa18
回退一个版本,清空暂存区,将已提交的内容的版本恢复到本地,本地的文件也将被恢复的版本替换
```



## branch

```properties
➜  gitDemo git:(work) git branch company   #创建分支
➜  gitDemo git:(work) git branch #查看所有分支
➜  gitDemo git:(work) git checkout company #切换分支
Switched to branch 'company'
➜  gitDemo git:(company) git status #查看当前git是哪个分支
On branch company
nothing to commit, working tree clean
➜  gitDemo git:(company)

➜  gitDemo git:(master) git branch -m work woro_relname #重命名分支
➜  gitDemo git:(master) git branch -d woro_relname #删除分支
➜  gitdemo git:(master) git branch -r  #查看远程分支 
```



## merge

```properties
➜  xiaoqiang git:(master) git checkout master  #切换到主分支
➜  xiaoqiang git:(master) git merge company   #合并 company分支

#如下合并文件冲突 
<<<<<<< HEAD
我是master分支
=======
我是company分支
>>>>>>> company

#手动删除这些提示出来的冲突文字， 然后在进行 git add 和 git  commit  就好，一般开发时候都是通过IDEA进行判对比后，然后在进行提交。
```





## log

```properties
#--graph     图形效果  
#--oneline   一行命令信息

➜  gitDemo git:(master) git log --oneline --decorate  -10 --graph

*   9744119 (HEAD -> master) Merge branch 'company'
|\
| * 366379a (company) company修改
* | 5e4ff50 master修改
* | edfb18a master修改
|/
* 190280e company分支
* dcb0636 (work) Revert "删除文件"
* 341fa18 删除文件
* 7a39e4a Revert "测试"
* ce2420c 恢复删除文件
* b25edc3 删除文件

#作者，过滤 
➜  gitDemo git:(master) git log --oneline
                                --author='' -10  
                                --grep='删除'                           
                                --before='2019-4-2' | --before='1 week'| --before='3 days'                               
```





## stash 

如果修改了A，B，C3个文件， 但是修改了A的这个文件我现在提交的时候， 不想和B，C文件一起提交，这个时候，就可以用到这个命令了。 **注意: 这个命令是按照你现在执行的阶段来的。**

```properties
#保存执行这个命令之前的所有操作
➜  gitDemo git:(master) ✗ git stash save '修改了a.txt'  

 #查看有几个，stash@{0}是他的标志，在这命令显示出来的
➜  gitDemo git:(master) git stash list 

#重新激活应用这个保存进度
➜  gitDemo git:(master) git stash apply stash@{0}  

#效果如下
➜  gitDemo git:(master) ✗ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   a.txt

no changes added to commit (use "git add" and/or "git commit -a")

# 删除这个保存进度
➜  gitDemo git:(master) ✗ git stash drop stash@{0}
➜  gitDemo git:(master) ✗ git stash pop
```





## gitignore

可以给每一个项目， 确定哪一些文件是可以忽略的。直接在项目的主目录创建.gitignore文件，添加过滤规则。

```properties
➜  gitDemo git:(master) cat .gitignore
*.log
➜  gitDemo git:(master) touch a.log
➜  gitDemo git:(master) git status
On branch master
nothing to commit, working tree clean
➜  gitDemo git:(master)
```



## remote

```properties
# origin 远程名称， 一般默认是这个
git remote add origin https://github.com/maizitoday/gitDemo.git
# 查看拉取和推送 
➜  gitdemo git:(master) git remote -v
origin	https://github.com/maizitoday/gitDemo.git (fetch)
origin	https://github.com/maizitoday/gitDemo.git (push)
```



## pull/push 

本地库推送到远程 ,如：github 

```properties
➜  gitdemo git init
➜  gitdemo git:(master) git remote add origin https://github.com/maizitoday/gitDemo.git

#需要先拉取下来， 如果直接推送会出现下面错误：
error: failed to push some refs to 'https://github.com/maizitoday/gitDemo.git'

#先拉取 
➜  gitdemo git:(master) git pull origin master

#在推送
➜  gitdemo git:(master) git push -u origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 277 bytes | 277.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/maizitoday/gitDemo.git
   52a6598..51415bf  master -> master
   
Branch 'master' set up to track remote branch 'master' from 'origin'.

-u 这个参数表示跟踪服务器上面的master分支 
#Branch 'master' set up to track remote branch 'master' from 'origin'.


#推送其他分支， 没有加-u 
➜  gitdemo git:(master) git push origin work
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'work' on GitHub by visiting:
remote:      https://github.com/maizitoday/gitDemo/pull/new/work
remote:
To https://github.com/maizitoday/gitDemo.git
 * [new branch]      work -> work
```



## remote workflow



### clone

克隆一个项目，取一个本地名字，然后修改文件，推送到服务端

```properties
➜  Desktop git clone https://github.com/maizitoday/gitDemo.git  maizi_git_demo
➜  maizi_git_demo git:(master) vi a.txt
➜  maizi_git_demo git:(master) ✗ git commit -am '提交'
[master c095f2e] 提交
 1 file changed, 1 insertion(+)
➜  maizi_git_demo git:(master) git remote
origin
➜  maizi_git_demo git:(master) git push origin master
```



### fetch

应用场景： 当我手动在github上面修改了[README.md]这个文件的时候， 我本地直接查看如下：

```properties
➜  maizi_git_demo git:(master) git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
➜  maizi_git_demo git:(master)
```

显示我的本地和远程是同步的： Your branch is up to date with 'origin/master'.所以多人协作的时候，就需要这个命令了。

```properties
# 调用 fetch 命令 提取一下最新代码(针对多人开发)
➜  maizi_git_demo git:(master) git fetch
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/maizitoday/gitDemo
   c095f2e..2ea8a42  master     -> origin/master
➜  maizi_git_demo git:(master)

#查看现在状态 
➜  maizi_git_demo git:(master) git status
On branch master
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean

#本地用户和服务器另一个用户提交的进行合并分支
➜  maizi_git_demo git:(master) git merge origin/master
Updating 2ea8a42..e062b17
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
 
 #现在查看已经同步数据了
➜  maizi_git_demo git:(master) git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```



### fork

就是把github上面的一个项目迁移到自己的github账号里面，用于自己基于这个项目来进行开发。



### pull-request

当你在github上面看大一个项目有bug，需要进行修改的时候，你就可以看到在github上面有一个pull-request，然后你创建一个pull-request, 选择好项目的版本和文件，然后选择你的版本和文件， 然后一步一步操作， 就可以对线上的项目进行BUG的修复了。



### collaborator

可以邀请github的账户，打开项目共享者列表， 一起来开发协助开发。 具体查看github上面的操作。 



# 本地文件上传到github

把现在已经生成的文件，上传到github，进行同步， 我这里直接把博客的文章上传到github中。 

```
➜  content git init
➜ content git:(master) ✗ git remote add origin https://github.com/maizitoday/myblogNotes.git

```







