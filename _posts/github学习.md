title: "github学习"
date: 2015-09-28 00:56:09
categories: 前端
tags: [github]
---


##git 及github使用学习

**总结点：**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分支合并后在git中操作`git branch`查看所有分支时分支还是会显示。并没有删除分支。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;merge操作时务必到主干上操作。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git reflog`跟`git reset --hard xxx`组合是很好的规避失误操作的方法。reflog查看所有仓库HEAD状态，reset回到那个状态。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在主干master上执行reset操作时，不会影响到已经打好的分支。

> 1、git init 初始化一个git项目

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在初始化的项目中有一个`.git`文件，可以被称为“附属于该仓库的工作树”。文件的操作都在工作树中进行，然后记录到仓库中去。

>2、git status 查看状态

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只要对git的工作树或者仓库操作，git status的现实结果就会发生变化。

>3、git add 向暂存区中添加文件
>暂存区属于仓库的范畴

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;假如只是在工作区操作文件，并不会在被计入git仓库的版本管理对象中。用status查看状态时只会出现在Untracked files里，而git add操作后，文件就进入了暂存区即仓库。

>4、git commit 保存仓库的历史记录
```
git commit --amend
//修改上一次提交的信息
git commit -am "message"
//add commit & 提交信息
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将当前暂存区的的文件实际保存到仓库的历史记录。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过这些记录我们就可以在工作区内复原文件。

>5、git log 查看提交的记录，也是保存过的仓库的记录。
```
git log --pretty=short
//不显示日期
git log -p README.md
//读取READMD.md文件提交记录，-p代表了前后差异
git log README.md 
//读取特定文件的提交记录
git log graph
//以图表形式查看分支
```

>6、git diff 查看更改前后的差别

```
git diff HEAD
//比较工作树和最新提交之间的差别。
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;git diff 可以查看工作树、暂存区、最新提交之间的差异。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;默认情况下`git diff`比较的是工作树和暂存区的区别，如果此分支在`git add`之后工作树跟暂存区区别是没有的，所以在add之后commit之前需要使用`git diff HEAD`比较工作树和最新提交之间的差别。

>7、git branch 显示分支一览表

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显示当前的仓库的所有分支。并且当前分支带有**星号**标志。

>8、git checkout -b feature-A  创建并切换到分支。
>git branch feature-A 创建分支
>git checkout feature-A 切换到feature-A分支

打分支
>9、git merge 合并分支

```
git merge --no-ff 分支
//在记录中记录本此合并
```

>10、git reset --hard xxx让仓库的HEAD、暂存区、和当前工作树回溯到指定版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;回溯之后查看log日志只能看到当前状态之前的日志，就看不到回溯之前的那些日志，如果回溯错了，想回到回溯之前的状态该怎么办？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只用`git reflog`操作就能看到包括回溯之前的所有状态，然后再配合`git reset --hard`命令回溯回去就好了。
git


>11、git rebase -i HEAD~2（num）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果觉得两次提交的内容和信息台贴近无需分成两次，就可以使用这个命令两xx条最近的HEAD合并成一条提交。
执行后回出现编辑框
```
pick 7aadfd "add feature"
pick 4tw436 "dsxxxx"
>>>>>>修改成
pick 7aadfd "add feature"
fixup 4tw436 "dsxxxx"
//意味着将第二条修改加到第一条去
```
##**第三方开发者协作**
>12、git remote add origin git@github.com:fromiris/git-test.git
>将远程仓库链接到本地仓库，并且讲远程仓库名字设置成origin

>13、git push -u origin master
>-u origin master 参数 表示将origin的master分支设置成本地master分支的upstream，这样以后pull时可以直接获取

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果是在本地开了一个分支并且推送到远程仓库也是这个语句。
`git push -u origin feature-D`
并且将远端的feature-D分支当成本地feature-D分支的upstream
这样之后在push只用`git push`就好了。


>1、git clone git@github.com:fromiris/git-test.git
>clone操作默认克隆远程仓库的master分支，并不包含其他分支。

```
git branch -a
//这个命令查看当前分支的相关信息，添加`-a`参数可以同时显示本地仓库和远程仓库的分支情况。
```
>2、checkout -b feature-D origin/feature-D


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;clone下仓库后并没有clone下其他分支，所以需要自己手动去在本地开分支，并且设定好远程仓库分支的来源。

>3、git pull origin feature-D
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从远端的featur-D分支拉取最新的数据，如果第一次push设置了
`git push -u origin feature-D`之后在这个分支上直接pull就好了。


##**github上的一些操作**
>1、Gist 

存放一些没必要放入一个仓库的代码片段，比如一些简单的组件js等。

>2、行号

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在具体仓库里的脚本文件，点击左侧可以在url中出现当前行的锚点。
![Alt text](./1443275342434.png)
这个功能适合人员之间进行讨论。

>3、在仓库中快速查找 `t`

![Alt text](./1443275543433.png)

>4、issue 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之前在issue里程序员一般都会探讨一些技术问题，追踪bug等，现在的issue可以做更多事情。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如开源项目规划排期，写博客
- [ ] 我的
- [x] 你得

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;提交信息可以跟issus的编号绑定。
![Alt text](./1443278089709.png)
这个#1就是这个开着的issus的编号，在代码commit的时候提交信息里出现#1，那么这个issue就会跟提交绑定。
![Alt text](./1443278184749.png)
同样，在提交信息里加上`close#1`就会把相关的issue关闭，省去了代码完成后还要一层层去找issue去关闭的现象。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**issus跟pull request**的编号是通用的。


>5、pull request

