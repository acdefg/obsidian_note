down:: [[git+action]]
[git如何设置多个用户名 • Worktile社区](https://worktile.com/kb/ask/221132.html)
[git怎么设置多个账号 • Worktile社区](https://worktile.com/kb/ask/237505.html)

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
git config user.name 'wenjtop'
git config user.email "1007131354.@qq.com"
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

git init 本地文件夹标记为 git 仓库

### git remote
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

