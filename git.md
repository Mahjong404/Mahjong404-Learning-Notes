### 本地

#### 工作区与暂存区
##### 1.初始化仓库
```bash
git init
```
##### 2.查看项目状态
```bash
git status
```
##### 3.添加文件到暂存区
```bash
git add <filename> # 添加特定文件到暂存区
git add . # 添加所有文件到暂存区
```

#### 本地仓库
##### 4.提交暂存区文件到本地仓库
```bash
git commit -m "commit message"
git commit -a -m "commit message"
```
##### 5.查看提交记录
```bash
git log --all --graph --oneline
```
##### 6.回滚
```bash
git reset --hard <commit_id>
```
##### 7.查看所有操作记录
```bash
git reflog
```

#### 基础分支操作
##### 8.查看/创建/删除/切换分支
```bash
git branch # 查看分支
git branch <branch_name> # 创建分支
git branch -d <branch_name> # 删除分支
git branch -D <branch_name> # 强制删除分支
git checkout <branch_name> # 切换分支
```
##### 9.合并分支
```bash
git merge <branch_name>
```
##### 10.查看差异
```bash
git diff
```
##### 11.变基
```bash
git rebase <target_branch_name>
```
##### 12.优选
```bash
git cherry-pick <commit_id>
```

##### 附：IDEA中文件颜色说明(Windows,还不准确)
 - 屎黄色：.gitignore
 - 白色：无修改
 - 红色：未添加到暂存区的文件
 - 绿色：已添加到暂存区的新文件
 - 蓝色：修改过的文件


### 远程
##### 13.添加/查看/删除远程仓库
```bash
git remote add <remote_name> <remote_url>
git remote -v
git remote remove <remote_name>
```
##### 14.推送本地分支到远程仓库
```bash
git push origin <remote_branch_name> <local_branch_name>
#推送本地分支到远程仓库
#例如 git push origin main main

git push --set-upstream origin <remote_branch_name>:<local_branch_name>
#将远端和本地的分支进行绑定，绑定后就不需要指定分支名称,即
git push origin
```
##### 15.克隆远程仓库到本地
```bash
git clone <remote_url>
#克隆远程仓库到本地
```
##### 16.抓取/合并/拉取远程仓库
```bash
git fetch origin #抓取：只获取但不合并远端分支，后面需要我们手动合并才能提交
git merge origin/<branch_name> #合并远端分支到本地

git pull origin #拉取：获取+合并
```

### 附：文件管理常用命令
```bash
ls -al #ls为查看文件，-a为显示所有文件，-l为显示详细信息
ls ../.. #查看上上级目录
pwd #查看当前目录位置
mkdir <dir_name> #创建文件夹
cd <dir_name> #进入文件夹
cd .. #返回上一级目录,cd .为返回当前目录
cd - #返回上一次进入的目录
cat <filename> #查看文件内容
vi <filename> #打开vim，编辑文件
```