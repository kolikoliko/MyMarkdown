# Git learning



## 基本操作

`mkdir name`创建仓库

`cd name`查找文件夹

`pwd`锁定文件夹路径

`git init`初始化git

`git add readme.txt `添加文件

`git commit -m "说明" `提交文件

`git log`查看提交历史（用于查看版本回退）

`git reflog`查看命令历史（用于后悔回退）

`git reset --hard HEAD^`版本回退

`git reset --hard 1094a`指定版本回退

`git status`查看git的状态

`cat readme.txt`查看文件内容

`git restore readme.txt`撤销所有工作区的操作

`git restore --staged readme.txt`将暂存区的退到工作区

`git rm readme.txt`删除文件



## 远程连接仓库

`git remote add origin git@github.com:path/repo-name.git`关联远程库

`orgin`是远程库的习惯命名

`git push -u origin master `第一次推送master分支的所有内容

`git push origin master`推送最新修改

`git clone git@server name:path/repo-name.git`克隆远程仓库



## 分支

`git branch`查看分支

`git branch <name>`创建分支

`git checkout <name>`切换分支

`git checkout -b <name>`或者`git switch -c <name>`创建并切换分支

`git merge <name>`合并分支到当前分支

`git branch -d <name>`删除分支

`checkout`和`switch`可以互换

`git stash`  储存工作区，（缓存工作区跳转到别的分支进行操作）

`git stash list`查看缓存地方

`git stash pop`或者`git stash apply`来恢复工作区（前者会把stash的内容也删掉）



## 标签

`git tag <name> <号码>`给commit的提交打上标签，无号码则是最新提交的commit

`git tag`查看标签

`git tag -a <tagname> -m "shuoming"`为标签添加说明

`git show <tagname>`可以看到说明文字

`git tag -d <tagname>`删除标签



## 一般流程

1. 右键需要的文件夹点击`git bash here`
2. 添加`.gitignore`和`readme.md`文件（`touch .gitignore`可创建）
3. `git init`初始化
4. `git add .`添加所有文件到暂存区
5. `git commit -m “state”`提交到master
6. 连接远程仓库，可按提示操作
7. 上传
