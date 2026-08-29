### Git 简介

Git 是一个免费的开源分布式版本控制系统，用于高效地处理项目中的各种变更。Git 允许多个开发者协作，对代码进行版本管理。

### 基本概念

- **Repository（仓库）**：存储项目代码的地方，可以是本地仓库或远程仓库。
- **Commit（提交）**：对代码的某个具体修改记录。
- **Branch（分支）**：开发中的一个独立分支，可以并行开发。
- **Merge（合并）**：将不同分支的修改合并到一起。
- **Clone（克隆）**：复制一个远程仓库到本地。
- **Pull（拉取）**：从远程仓库获取最新的代码并合并到本地。
- **Push（推送）**：将本地的提交上传到远程仓库。

### 基本操作

#### 1. 安装 Git

从 [Git 官网](https://git-scm.com/) 下载并安装适用于你操作系统的 Git 版本。

#### 2. 配置 Git

安装完 Git 后，需要进行一些基本配置：

```bash
git config --global user.name "你的姓名" 
git config --global user.email "你的邮箱"
```

#### 3. 创建仓库

在一个项目文件夹中初始化一个新的 Git 仓库：

```bash
git init
```

#### 4. 克隆远程仓库

从远程仓库克隆一个项目到本地：

```bash
git clone 仓库地址
```
#### 5. 查看仓库状态

查看当前工作目录的状态，了解有哪些文件被修改或未被跟踪：

```bash
git status
```

#### 6. 添加文件到暂存区

将修改的文件添加到暂存区（准备提交）：

```bash
git add 文件名 # 或添加所有修改的文件 git add .
```

#### 7. 提交修改

将暂存区的修改提交到本地仓库：

```bash
git commit -m "提交信息"
```

#### 8. 查看提交历史

查看仓库的提交历史：

```bash
git log
```

#### 9. 创建分支

创建一个新分支：

```bash
git branch 分支名
```

#### 10. 切换分支

切换到指定分支：

```bash
git checkout 分支名
```

#### 11. 合并分支

将其他分支的修改合并到当前分支：

```bash
git merge 分支名
```

#### 12. 推送到远程仓库

将本地的提交推送到远程仓库：

```bash
git push origin 分支名
```

#### 13. 拉取远程修改

从远程仓库拉取最新的修改并合并到本地：

```bash
git pull
```

### 关联远程分支
在拿到项目代码后,有时没有和git仓库进行关联,这时就需要我们手动关联;

#### 1. 首先关联仓库
```bash
git remote add origin <远程仓库的URL>
```
#### 2.将本地分支与远程分支关联
```bash
#远程有分支
git branch --set-upstream-to=origin/<分支名> <分支名>
#需要新建分支

```

### 本地分支覆盖
```bash
# 切换到要保留的分支
git checkout master
# 强制推送本地 `master` 分支到远程 `main` 分支
git push origin master:main --force
步骤 3：验证推送状态
git log origin/main
```