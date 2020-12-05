# Git学习笔记

## Git的两大特点

1. 版本控制：可以解决多人同时开发的代码问题，也可以解决找回历史代码的问题。
2. 分布式：Git是分布式版本控制系统，同一个Git仓库，可以分布到不同的机器上。

## Git安装和创建本地版本库

1. 安装git 

   ```
   sudo apt install git
   ```

2. 创建一个文件夹保存本地版本库

   ```
   mkdir git
   cd git
   ```

3. 在git文件夹下创建本地版本库

   ```
   git init
   ```

   

## Git常用命令

1. 创建版本

```
git add filename
git commit -m "version_name"
```

2. 查看版本

```
git log  //完整显示
git log --pretty=oneline  //简短显示
git log --graph --pretty=oneline  //查看分支树简短显示
```

3. 回退到某一个版本
   1. git reset --hard HEAD^,尖括号的个数表示回退的版本数
   2. git reset --hard HEAD～[number]，number表示回退的版本数
   3. git reset --hard 版本号,表示回退到版本号对应的版本

```
	git reset --hard HEAD^
	git reset --hard HEAD~1
	git reset --hard e7c23c79a52257539c22d91eaeead9da85e32047
```

 4. 查看版本的提交记录

    ```
    git reflog
    ```

	5. 查看工作状态

    ```
    git status
    ```

	6. 丢弃工作区中对文件的修改

    ```
    git checkout -- filename
    或
    git restore filename
    ```

	7. 丢弃文件在暂存区中的改动

    ```
    git reset HEAD  filename
    git checkout -- filename
    或者
    git restore --staged filename
    git restore filename
    ```

	8. 对比文件的不同

    ```
    git diff HEAD HEAD^ -- filename  //表示前一个版本与当前版本的不同
    git diff HEAD -- file //表示对比工作区与当前版本文件的不同
    ```

	9. 删除文件

    ```
    rm filename
    git rm filename
    git commit -m "version_info"
    ```

## 工作区与暂存区

1. 工作区（Working Directory），电脑中的git版本库目录.
2. 版本库（Repository）,工作区中的.git隐藏目录，保存着暂存区和自动创建的分支master，以及指向master的指针HEAD.
3. 暂存区（index或stage）,在.git隐藏目录下
4. 在往版本库添加文件时，分两步执行：
   1. 用git add 把文件添加进暂存区
   2. 用git commit把暂存区中的文件全部提交到当前分支
5. git管理的文件修改，只会提交暂存区的修改来创建版本

## 分支

1. 查看分支

   ```
   git branch
   ```

2. 创建分支

   ```
   git branch <name>
   ```

3. 切换分支

   ```
   git checkout <name>
   ```

4. 创建并切换分支

   ```
   git checkout -b <name>
   ```

5. 合并分支

   ```
   git merge <name>  //合并指定分支到当前分支
   git merge --no-ff -m "version_info" <name>  //禁止快速合并
   ```

6. 删除分支

   ```
   git branch -d <name>
   ```

7. 保存工作状态

   ```
   git stash
   ```

8. 查看保存的工作状态

   ```
   git stash list
   ```

9. 恢复工作状态

   ```
   git stash pop
   ```

   

## Github使用

1. 克隆项目

```
git clone <ssh key>
```

2. 推送分支

   ```
   git push origin <branch name>
   ```

3. 跟踪远程分支

   ```
   git branch --set-upstream-to=origin/<branch name> <branch name>
   ```

4. 拉取远程分支的代码

   ```
   git pull origin mybranch
   ```
5. 设置远程仓库的地址
	```
	git remote set-url origin <repositories name>
	```
   
