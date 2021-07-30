# Git命令

## 1. 基础命令

### 1.1 初始化仓库

通过`git init`建立仓库，目录下为多一个`.git`目录，这是Git跟踪管理版本的库。

```
git init [-q | --quiet] [--bare] [--template=<template_directory>]
	  [--separate-git-dir <git dir>] [--object-format=<format>]
	  [-b <branch-name> | --initial-branch=<branch-name>]
	  [--shared[=<permissions>]] [directory]
```

**选项说明:**

**-q**或**--quiet**

​	仅打印错误和警告消息；所有其他输出将不会显示。

**--bare**

​	创建一个纯仓库。如果未设置 `GIT_DIR` 环境变量，则将其设置为当前工作目录。

**--object-format=<format>**

​	为仓库指定对象格式。可选值为**sha1**和**sha256**。默认值是**sha1**。

**--template=<template_directory>**

​	指定要使用模板的目录。

**--separate-git-dir=<git dir>**

​	并不将存储库初始化至 `$GIT_DIR` 或 `./.git/` 目录，而是在其中创建一个包含实际存储库路径的文本文件。此文件作为连接到仓库的 Git 符号链接，其与文件系统无关。

​	如果为重新初始化操作，则将存储库移动到指定的路径。

**-b <branch-name**或**--initial-branch=<branch-name>**

​	在新创建的仓库中为初始分支指定名称。如果没有指定，则使用默认名称：`master`。

**--shared[=(false|true|umask|group|all|world|everybody|0xxx)]**

指定 Git 存储库在多个用户之间共享。这允许属于同一组的用户推送到该存储库。指定时，将设置配置变量 "core.sharedRepository"，以便使用请求的权限创建 `$GIT_DIR` 下的文件和目录。未指定时，Git 将使用 umask(2) 返回的权限。

此选项可以有以下值，如果未给定值，则默认为 *group*：

- *umask*（或 *false*）

  使用 umask(2) 返回的权限。未指定 `--shared` 时，此为默认值。

- *group*（或 *true*）

  使存储库组可写（并且 g+sx，因为 git 组可能不是所有用户的主要组）。这用于放宽原本安全的 umask(2) 值的权限。请注意，umask 仍然适用于其他权限位（例如，如果 umask 为 *0022*，则使用 *group* 不会删除其他（非组）用户的读取特权）。有关如何精确指定存储库权限的信息，请参见 *0xxx*。

- *all* （或 *world* 或 *everybody*）

  与使用 *group* 选项相同，但使存储库对所有用户可读。

- *0xxx*

  *0xxx* 是一个八进制数，每个文件的模式均为 *0xxx*。' 0xxx' 将覆盖用户的 umask(2) 值（不仅像 *group 和 'all* 那样放宽权限）。*0640* 将创建一个组可读取但不能组写入且其他用户无法访问的存储库。*0660* 将创建当前用户和组可读可写但其他用户无法访问的存储库。

### 1.2 Clone

通过`git clone`拷贝仓库到新的目录。

```
git clone [--template=<template_directory>]
	  [-l] [-s] [--no-hardlinks] [-q] [-n] [--bare] [--mirror]
	  [-o <name>] [-b <name>] [-u <upload-pack>] [--reference <repository>]
	  [--dissociate] [--separate-git-dir <git dir>]
	  [--depth <depth>] [--[no-]single-branch] [--no-tags]
	  [--recurse-submodules[=<pathspec>]] [--[no-]shallow-submodules]
	  [--[no-]remote-submodules] [--jobs <n>] [--sparse] [--[no-]reject-shallow]
	  [--filter=<filter>] [--] <repository>
	  [<directory>]
```

**常用选项说明:**

**-b <name>**或**--branch <name>**

​	在非裸仓库中，将新创建的HEAD指向`name`分支。

**--depth <depth>**

​	克隆仓库，到指定提交次数的历史记录截止。

[git clone文档](https://git-scm.com/docs/git-clone)

## 2. SETUP

````shell
git config --global user.name "[firstname lastname]"		# 设置用户名
git config --global user.email "[valid-email]"					# 设置邮箱地址
git config --global color.ui auto												# 设置命令行颜色
````

## 3. STAGE&SNAPSHOT

````shell
git status															# 显示工作目录下修改的文件
git add [file]													# 添加文件到暂存区(索引区)
git reset [file]												# 将文件撤出暂存区，并保留修改内容
git diff 																# 查看工作区文件的变化
git diff --stage												# 查看暂存区文件的变化
git commit -m "[descriptive message]"		# 提交暂存区内容
````

## 4. 分支管理

### 4.1 查看分支

````shell
git branch 															# 列出本地分支，*标记的为当前活跃分支
git branch -la													# 查看本地和远程分支
````

### 4.2 新建分支

````shell
git branch [branch-name]								# 创建新的分支
git checkout -b [branch-name]						# 创建新的分支并切换到新分支
````

### 4.3 切换分支

````shell
git checkout 														# 切换分支，并检出到当前工作目录
````

### 4.4 删除分支

````shell
# 删除本地分支
git branch -d <branchname>
# 或者
git branch --delete <branchname>

# 强制删除, -D == --delete --force
git branch -D <branchname>

# 删除远程分支
git push origin --delete <branchname>
````

### 4.5 重命名分支

````shell
git branch -m <oldbranch> <newbranch>
# 或者
git branch --move

# 强制删除, -M == --move --force
git branch -M <oldbranch> <newbranch>
````

### 4.6 合并分支

````shell
git merge <branch>			# 合并指定的分支到当前分支
````



