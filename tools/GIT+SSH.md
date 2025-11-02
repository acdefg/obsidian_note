down:: [[git+action]]
[git如何设置多个用户名 • Worktile社区](https://worktile.com/kb/ask/221132.html)
[git怎么设置多个账号 • Worktile社区](https://worktile.com/kb/ask/237505.html)

### 新机器安装
[Github——git本地仓库建立与远程连接（最详细清晰版本！附简化步骤与常见错误）\_将本地仓库与远程仓库关联-CSDN博客](https://blog.csdn.net/qq_29493173/article/details/113094143)
## 常用指令
git clone git add git commit git push git pull git init git log

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410311628284.png?token=ALRC6IUMHYMCRMMKSIGBROLHEM75W)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410311649724.png?token=ALRC6IWP54ZYMPMRNQPLR4LHENCO4)

### 版本管理
git branch：创建分支
git checkout：切换分支
git merge：分支混合

### 远程
git remote git fetch git diff git pull
git fetch：从远程拉取到本地，但不会修改本地文件
git diff：查看本地和远程仓库的区别，git fetch 之后使用

### tips
vscode 使用 git graph 管理 git 分支非常方便
husky 可以管理 commit message

rebase checkout

## git 详细

### Git 安装

```text
# ubuntu 安装
sudo apt update
sudo apt install git

# 其它系统可以去官网下载包安装
```

### git 配置（git config）

```text
# 给git配置用户名
git config --global user.name 'wenjtop'
git config --global user.email "1007131354.@qq.com" 
# --global 全局信息，所有仓库都生效。不加只对当前仓库生效。
# --system 系统配置，所有用户都生效。很少使用。

git config --global credential store # 存储配置
git config --global --list           # 查看当前仓库配置信息
```

只配置当前仓库的信息，用于多用户配置时
```
git config user.name 'xxxx'
git config user.email "xxxx"
```

查看配置：
```
git config user.name
git config user.email

git config -l #查看详细信息
```

取消 config 设置：
```
git config unset user.name
git config --global --unset user.name
git config --global --unset http.proxy #取消代理
```

### git 仓库建立
git clone
```
# 格式：git clone -b <分支名> <URL>
git clone -b rsdmike-patch-1 https://gitee.com/EdgexFoundry/edgex-examples.git

#不考虑分支
git clone https://xxxxx
git clone git@xxxx    #ssh
```

#### tips
```
git init  #本地文件夹标记为 git 仓库
git remote add origin git@github.com:wenjtop/RepositoryTest.git
git branch -M main   #-M 修改名称
git push -u origin main  #-u 设置默认 之后可以直接用 git push做同样的操作
git commit -a -m "first commit" # 可以同时执行add和commit操作
git add .   #添加当前目录下所有文件
```

### git remote

```text
git remote -v                                           # 显示所有远程仓库
git remote add origin https://github.com/user/repo.git  # 添加远程版本库
git remote rename origin new-origin                     # 修改仓库名
git remote remove new-origin                            # 删除远程仓库
git remote set-url origin https://github.com/user/new-repo.git   # 修改指定远程仓库的 URL
git remote show origin                                  # 显示某个远程仓库的信息
```

列出远程仓库：
```
git remote
git remote -v
```
-v 选项会显示远程仓库的 URL 地址。

**添加远程仓库：**
```
git remote add [别名] [URL]
```
这里 [别名] 是你给远程仓库设置的本地引用名（默认为 origin），[URL] 是远程仓库的地址。

移除远程仓库：
```
git remote remove [别名]
```
或简写为
```
git remote rm [别名]
```

**修改远程仓库的 URL：**
```
git remote set-url [别名] [新的 URL]
```
如果要修改远程仓库的推送 URL（push URL），可以使用 --push 选项：
```
git remote set-url --push [别名] [新的推送 URL]
```

显示远程仓库的详细信息：
```
git remote show [别名]
```
这个命令会显示指定远程仓库的详细信息，包括远程跟踪分支和 URL 地址等。
  
原文链接：https://blog.csdn.net/2201_75439183/article/details/142378739
更多：[https://zhuanlan.zhihu.com/p/694960607](https://zhuanlan.zhihu.com/p/694960607)

### git 分支

```text
git branch               # 查看在哪个分支上
git branch 分支名         # 创建分支
git switch 新分支名       # 却换到新分支名上，新方法
git checkout -b 新分支名  # 创建新分支并，切换到新分支名上，旧方法
git branch -d 分支名      # 删除以合并的分支，-D强制删除
git merge dev 分支名      # 当面分支为目标分支，后面为需要合并的分支
```

#### git branch

```
git branch   # 查看在哪个分支上
git branch < branchName > # 创建新的本地分支，但是不会进行切换
git branch -vv #查看分支详细信息
git branch -r # 查看远程分支
git branch -a # 查看所有分支
git branch --set-[upstream]-to=origin/< branch > feture-test
#这个命令用于将本地分支与远程分支建立连接。< branch >是远程分支名，feture-test是本地分支名
git branch -m old new / git branch -M old new # 重命名分支
git branch -d branchname / git branch -D branchname # 删除本地分支
git branch -d -r branchname # 删除远程分支
```

#### git checkout

**切换本地分支**
```git
#切换到”branchname“分支，注意是本地分支。
git checkout branchname  #（切换本地分支）
```

**切换远程分支**
该命令可以将远程仓库里指定的分支拉取到本地，并在本地创建一个分支与指定远程仓库分支关联起来。并切换到新建的本地分支中。
```
git checkout -b 本地分支名 origin/远程分支名
#example
git checkout -b happy/0817_test  # orgin/main origin对应远程仓库 main对应分支名
```

**放弃修改**
```
git checkout             #放弃所有工作区的修改
git checkout – filename  #放弃对指定文件的修改
git checkout -f          #放弃工作区和暂存区的所有修改
```

### git switch

git switch 切换分支
  命令：`git switch <branchName>`
  举例：`git switch testBranch`

**git switch 创建一个新分支并切换到该新分支**
  命令：`git switch -c <branchName>`
  举例：`git switch -c test3`
  tips：如果分支已存在，git 会报：fatal: A branch named ‘test2’ already exists. 我们可以使用 git branch 查看当前本地有哪些分支。

**git switch 以一个提交 commit 来创建一个分支**
  命令：`git switch -c test3 <commit>`
  举例：`git switch -c test3 e053cf128d2ad9d35e2f94878569596fb32f4306`

git switch 以一个 tag 来创建一个分支
  命令：`git switch -c <newBranchName> <tagName>`
  举例：`git switch -c testcopytagbr testcopytag1`

**git switch 切换到某一个 commit 但是不创建新的分支，可以查看这个记录是的修改情况**
  命令：`git switch --detach <commit>`
  举例：`git switch --detach a434bda`
  tips：如果我们不在查看这个历史记录的情况，只需要 git switch 到那个分支就可以了。如果切换到以前的某个记录了，看不到后面提交的记录了，可以使用 git reflog 查看所有修改记录。

**远程有而本地没有的分支，而如果要从远程分支建一个同名的本地分支，并且关联远程分支**
  命令：`git switch <branchName>`
  举例：`git switch testmaster`
  tips：这里我们也可以理解为拉取远程分支到本地，并建立远程分支和本地分支的关联关系

git switch 切换到上一个切换的分支
  命令：`git switch -`
  tips：如我们由 test1 分支切换到 test2 分支，而后我们使用此命令就会切换到 test1 分支去，再使用此命令又会切换到 test2 分支来，即一直使用这个命令，就会在 test1 和 test2 分支来回切换。如果上一个切换的分支被删除了，那么会报：fatal: invalid reference: @{-1}

git switch 创建一个没有任何提交记录的分支，删除所有跟踪的文件
  命令：`git switch --orphan <branchName>`
  举例：`git switch --orphan testmaster3`
                       
原文链接：https://blog.csdn.net/u012273398/article/details/139860334

### 分支合并（merge 和 rebase）
#### 工作方式：
-   当你在目标分支（如 `main`）上运行 `git merge feature-branch` 时，Git 会将 `feature-branch` 分支的更改合并到当前分支。
-   如果两个分支没有冲突，Git 会自动创建一个新的合并提交，将这两个分支的历史记录结合在一起。
-   如果存在冲突，Git 会要求你手动解决冲突，然后创建合并提交。
`git rebase` 是另一种合并更改的方式，但它通过重新应用提交来改变历史记录，使提交历史更加线性。
原文：[尽量“手撕”代码系列 - 飞书云文档](https://dwexzknzsh8.feishu.cn/docx/VkYud3H0zoDTrrxNX5lce0S4nDh)

### git 版本回退

这篇讲的比较好，但还没整理
[亡羊补牢，一文讲清各种场景下GIT如何回退\_git 回退-CSDN博客](https://blog.csdn.net/u011709538/article/details/139264216)

#### 基本方法
`git log` 查看版本号
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410311855060.png?token=ALRC6IUAXYIZ6IPPVJMW2RLHENRHG)

`git reset` 删除提交记录，可能修改工作区和暂存区
`git revert` 创建新的提交记录
`git checkout` 执行该命令后，会将工作区和暂存区的内容恢复到指定版本
使用 `git reflog` 命令查看历史操作记录并回退版本：  
```
git reflog  
git reset  
```
`git reflog` 命令可以查看到所有的操作记录，包括切换分支、提交等。可以根据 `git reflog` 输出的信息选择需要回退的版本，然后使用 `git reset` 命令回退版本。
使用 `git cherry-pick` 命令选择特定版本合并到当前分支：  
```
git cherry-pick  版本号
```
可以通过 `git log` 命令查看到。执行该命令后，会将指定版本的提交合并到当前分支，相当于回退到了指定版本。

#### 详细解释
`git reset --hard 目标版本号` 
强制回退到某个版本，并且删除之后的版本
`git push -f` 因为当前版本落后于远程版本，使用`-f`上传

`git revert 目标版本号`
原理： git revert 是用于“反做”某一个版本，以达到撤销该版本的修改的目的。比如，我们 commit 了三个版本（版本一、版本二、 版本三），突然发现版本二不行（如：有 bug），想要撤销版本二，但又不想影响撤销版本三的提交，就可以用 git revert 命令来反做版本二，生成新的版本四，这个版本四里会保留版本三的东西，但撤销了版本二的东西。如下图所示：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410311810895.png?token=ALRC6ITMFT5TXYDAYS5EFBLHENL7C)

原文链接：https://blog.csdn.net/2401_84910962/article/details/138935767

### 安全地回退到历史版本而不修改历史记录
**操作步骤：**
1.  **创建临时分支**

```bash
git checkout -b temp-version2 <commit-hash-of-version2>
```

这将在版本2的提交哈希处创建新分支
2.  **强制推送到远程进行GitHub Actions测试**

```bash
git push origin temp-version2 --force-with-lease
```

3.  **通过GitHub界面触发Actions**  
    在仓库的Actions标签页选择临时分支运行工作流
    
4.  **恢复原始状态**  
    测试完成后切换回原分支：

```bash
git checkout main
git branch -D temp-version2
git push origin --delete temp-version2
```

**完整命令示例：**

```
# 查找版本2的提交ID
git log --oneline

# 假设版本2提交哈希为abc1234
git checkout -b test-env abc1234
git push origin test-env

# 在GitHub界面配置Actions触发条件为test-env分支的推送事件

# 测试完成后清理
git checkout main
git branch -D test-env
git push origin --delete test-env
```