## 创建

```bash
# 初始化本地仓库 
git init

# 将远仓库的内容克隆到本地
git clone [remote url]
```



## 提交

```bash
# 查看本地库状态
git status

# 显示暂存区和工作区的差异
git diff [file]

# 显示暂存区和上一次提交的差异
git diff --cached [file]
git diff --staged [file]

# 显示两次提交之间的差异
git diff [first-branch]...[second-branch]

# 添加文件到暂存区 
git add [file1] [file2] ...  

# 添加目录到暂存区
git add [dir]

# 添加所有文件到暂存区
git add .

# 提交到本地库 
git commit -m [message]
git commit [file1] [file2] ... -m [message]

# 修改注释
git commit --amend

# 直接提交，跳过add
git commit -a

# 设置提交时的信息
git config --global user.name 'username'
git config --global user.email 'email'
```



## 日志

```bash
# 查看历史记录 
git reflog

# 查看版本详细信息 
git log

# 以列表形式查看指定文件的修改记录
git blame [file]
```



## 回退

```bash
# 撤销commit
git reset --soft HEAD^

# 撤销add
git reset --mixed HEAD^

# 回到上一次commit  
git reset --hard HEAD^

# 版本穿梭          
git reset --hard <logId>
```



## 删除

```bash
# 从暂存区和工作区中删除
git rm -f <file>

# 从暂存区中移除，但保留在工作区
git rm --cached <file>
```



## 远程

```bash
# 显示所有远程仓库
git remote -v

# 显示某个远程仓库
git remote show <remote>

# 添加远程仓库
git remote add <remote> <url>

# 删除远程仓库
git remote rm <name>

# 修改仓库名
git remote rename <oldName> <newName>
```



## 分支

```bash
# 查看本地所有分支 
git branch                           

# 查看远程所有分支
git branch -r                         

# 查看分支信息，附带版本 
git branch -v                        

# 查看本地和远程所有分支 
git branch -a                        

# 创建分支，停留在当前分支
git branch <newBranch>                    

# 创建并切换到新分支
git branch -f <branch>                

# 删除本地分支
git branch -d\D <localBranch>

# 将本地分支推送至远程分支
git branch <localBranch>:<remoteBranch>       

# 删除远程分支
git branch :<remoteBranch>

# 重命名分支，如是远程分支，则先删除后推送新分支
git branch -m\M <oldBranch> <newBranch>

# 切换分支
git checkout <branch>      
```



## 拉取合并

```bash
# 通过远程别名拉取远端仓库数据
git fetch <shortName>

# 合并到本地
git merge [branch]

# 从远程拉取并合并到本地某个分支
git pull <shortName> [<remoteBranch>:<localBranch>]
```



## 推送

```bash
# 推送本地分支上的内容到远程仓库 
git push <remote> [localBranch:]<remoteBranch>

# 强制推送
git push --force <remote> <branch>

# 删除远程分支 
git push origin --delete <remoteBranch>

# -u指定默认主机，以后可直接使用git push
git push -u <remote> <remoteBranch>
```
