### 安装

在安装完git之后，首先要配置username和email

> git config --global user.name "***"

> git config --global user.email "***@gmail.com"

可以使用git config --list查看已设配置。

### 初始化、克隆、添加、提交、推送远程

初始化
>git init

克隆远程URL
>git clone httP://****.git

添加更改文件
>git add A.txt/--all/.

提交到本地仓库并注释
>git commit -m "init"

推送到远程master
>git push origin master

### 分支操作

查询当前分支
>git branch

创建新分支
>git branch test

切换分支
>git checkout test

删除分支（必须先切换到其它分支下）
>git branch -d test

克隆远程分支
>git branch --track test

拉取远程分支
>git fetch

强制拉取远程分支更新到本地(如果和本地有冲突，本地先提交,相当于fetch+merge)
>git pull

合并分支test到master,如果有冲突，先解决冲突，手动添加到暂存区，然后commit
>git checkout master

>git merge test
 

### 其它命令

查看分支状态
>git status

查看提交日志
>git log

查看某个文件提交日志
>git log -p filename

本地删除未提交，还原命令
>git checkout --filename

查询git命令
>git

查询git某个指令详情
>git *** -v


### 忽略文件

在git工作区中有个.gitignore的文件

单独过滤一个文件：db.json

单独过滤后缀名文件：*.log

单独过滤某个目录：public/

单独过滤某个前缀为.deploy的目录：.deploy*/

### git文件状态

modified:被修改过的文件
Staged：通过git add 添加到暂存区
commited：通过git commit提交到本地仓库的文件


[SourceTreeURL]:https://www.sourcetreeapp.com/
[gitURL]:https://git-scm.com/downloads