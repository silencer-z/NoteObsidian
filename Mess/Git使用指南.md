### 已有文件如何上传到github中
1. 下载git，并设置user.email和user.name
2. 将内容提交到本地仓库 git commit
3. 登录github并创建对应仓库，最好为空，不要加入什么readme
4. 将远程仓库与本地仓库链接起来
5. 同步拉取远程仓库，合并保留本地的，然后再推送至远程
![[Pasted image 20240327163030.png]]
### 常用指令

```bash
本地初始化代码
git init    初始化本地工作区和本地仓库，产生一个master分支
git clone  <remote url> 从远程仓库拉取到本地，并初始化本地仓库
git remote add origin <remote url> 添加远程仓库
完成链路回环
git add -a
git pull origin <remote branch name>:<local branch n ame> 拉取到本地仓库
git push origin <remote branch name> 推送到远程分支
分支管理
git branch 列出本地已经存在的分支，并且当前分支会用*标记
git branch -r 查看远程版本库的分支列表
git branch -a 查看所有分支列表（包括本地和远程，remotes/开头的表示远程分支）
git branch -v 查看一个分支的最后一次提交
git branch --merged  查看哪些分支已经合并到当前分支
git branch --no-merged 查看所有未合并工作的分支


git branch <branch name> 新建一个本地分支
git branch -d <branch name> 删除本地分支
git branch -D <branch name> 强制删除本地分支
git checkout <branch name> 切换一个本地存在的分支
git checkout -b <branch name> 新建一个本地分支并切换

git branch --set-upstream-to origin/<branch name> 关联本地分支到远程分支

git push --set-upstream origin <branch name> 
git push origin :<branch name> 删除远程分支，冒号表示删除
git push origin --delete <branch-name> 删除远程分支


git push -u origin <remote branch name> 指定默认远程分支并推送
```

### token生成并绑定
在使用git修改仓库设置时需要绑定token
设置-> 开发者设置里找到令牌，生成符合自己需求的令牌，并粘贴
为了避免每次都输入token，可以将远程链接设置为如下形式
```shell
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
```
## 版本更新策略
>>https://nvie.com/posts/a-successful-git-branching-model/

![[Pasted image 20240407105025.png]]
#### master分支
master分支是主分支，包含了已经发布到生产环境的稳定，可靠版本的代码。一般情况下，master分支应该只用于发布新版本，而不应该直接修改或提交新的功能。

#### develop分支
develop分支是开发分支，包含了当前正在进行的所有功能和任务。所有新功能开发、改进、优化等都应该从此分支开始，并最终合并回此分支。

#### feature分支
feature分支是从develop分支创建的分支，通常用于开发新功能。每个新功能都应该从develop分支开始，并在一个独立的feature分支上进行开发工作。一旦新功能得到完全实现、测试并且可靠，该分支就会被合并回develop分支。

#### release分支
release分支是从develop分支创建的分支，通常用于为即将发布的版本做准备工作。在此分支上可以进行最终的测试、修复bug、检查文档等操作，以确保发布版本的质量。一旦准备工作完成并且得到完全测试，该分支就会被合并回master分支，并作为新的发布版本。并将该分支合并回develop分支，以便后续的开发工作。

#### hotfix分支
hotfix分支是从master分支创建的分支，用于在生产环境中紧急修复问题。修复完毕后，该分支将会被合并回master和develop分支。