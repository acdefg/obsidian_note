### 安装和设置git
1. 安装git
```shell
sudo apt-get install git
```
2. 设置账户
```
git config --global user.name "yourname" 
git config --global user.email "youremail"
```
替换的时候不用保留引号
3. 检查

```shell
git config --global user.name "yourname" 
git config --global user.email "youremail"
```

```shell
git config --global user.name "acdefg" 
git config --global user.email "ljc512@outllok.com"
```

替换的时候不用保留引号
3. 检查
```shell  
git config user.name 
git config user.email
```
Or

```shell
git config --list
```
会用 vim 打开一个文件，写着上面的信息

4. 生成公匙

```shell
ssh-keygen -t rsa -C "youremail" 
```
提示的地方直接按 Enter

```shell
cat  ~/.ssh/id_rsa.pub
```
或者

```shell
gedit ~/.ssh/id_rsa.pub
```
查看公匙，复制公匙

5. 添加到 github 后台
打开 github 个人主页，点击左上角头像，打开 setting，选择 SSH
![](https://s2.loli.net/2022/05/03/fqpDbIJ81S5ej9W.png)
![](https://s2.loli.net/2022/05/03/McjYFSmEKyhzwg6.png)
添加，确认一下就可以了

### git仓库设置
#### 本地
在本地新建目录/原有目录中输入

```shell
git init
```
通过`git add files`，添加文件或者文件夹到版本库 option：files
`git status`可以查看跟踪状态
`git commit -m "description"`  上传到版本库option：description
`git commit -am "description"` git commit -m用于提交暂存区的文件，git commit -am用于提交跟踪过的文件(obsidian上传推荐用这个)。
`git log` 查看上传记录
	会给出commit id和上传时写的description
`git reset --hard HEAD^` 返回上一个版本
	--hard参数之后再解释
	上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`

#### 远程仓库

```shell
git remote add origin git@github.com:acdefg/obsidian_note.git
git branch -M main
git push -u origin main
```
`git branch -M main` 改分支名字，可以不改，默认是master，第三句变为`git push -u origin master`

### obsidian使用git备份
安装obsidian_git插件，这个只是一个自动上下拉的插件，得先按前面的方法配置好远程仓库和本地仓库的链接。
还可以用gitee：git commit -m用于提交暂存区的文件，git commit -am用于提交跟踪过的文件。

### 删除操作
删除掉已经commit的文件，因为太大了一直导致push失败
参考链接：先看1，再看2
1. [Git清理commit中历史提交的大文件 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1536481)
2. [记一次删除Git记录中的大文件的过程-HollisChuang's Blog](https://www.hollischuang.com/archives/1708)---看这个就行了


### 解决冲突
[git如何解决冲突？_蛞蝓不孤寡的博客-CSDN博客_git怎么处理冲突](https://blog.csdn.net/fish_skyyyy/article/details/119539747)
应该可以设置一下 merge 规则
```shell
git add [冲突文件]  （取本地）
git rm [冲突文件]  （取远程端）
git commit -m "2023.10.29"
git pull
```

一些教程记录
[Fetching Title#15ws](https://blog.csdn.net/qq_34842671/article/details/70916587)  ---win下git安装教程
[Git: ‘LF will be replaced by CRLF the next time Git touches it‘ 问题解决与思考_Babylonxun的博客-CSDN博客](https://blog.csdn.net/Babylonxun/article/details/126598477)
[Fetching Title#ty7c](https://www.liaoxuefeng.com/wiki/896043488029600) --git教程  --廖雪峰

[详解gitignore的使用方法，让你尽情使用git add . - 知乎](https://zhuanlan.zhihu.com/p/264995020)

测试一下 obsidian_git

### 使用过程的问题 log
bad objects：
[BitBucket Git Error: did not send all necessary objects - Stack Overflow](https://stackoverflow.com/questions/8788975/bitbucket-git-error-did-not-send-all-necessary-objects/70957667#70957667)

### 其他用户修改上传
1. 下载仓库，初始化仓库 git init
2. 添加远程仓库（不要用 https，用 git@）`git remote add origin git@github.com:acdefg/obsidian_note.git`  
git branch -M main
3. 给仓库添加共同协作者
4. 给仓库添加 ssh key（不确定有没有影响）

### git 将本地修改后的文件提交到远程
**一 提交代码到远程 **

初始化版本库：
git init
添加文件到版本库（只是添加到缓存区），. 代表添加文件夹下所有文件
git add .
把添加的文件提交到版本库，并填写提交备注(必不可少)
git commit -m “update readme”
把本地库与远程库关联（如果已经有 origin 关联则可以忽略）
git remote add origin 你的远程库地址
推送（提交）代码：
git push <远程主机名> <本地分支名>:<远程分支名>
如：git push origin(主机名) master(本地分支名):master(远程分支名)
若需要提交指定文件：
1. 先查看更改的文件：
git status 查看仓库状态
2. 指定提交文件
git add src/components/文件名 添加需要提交的文件名（加路径–参考 git status 打印出来的文件路径）
3.
git stash -u -k 忽略其他文件，把现修改的隐藏起来，这样提交的时候就不会提交未被 add 的文件
4.git commit -m “哪里做了修改可写入…”
5.git pull 拉取合并
6.git push 推送到远程仓库
7.git stash pop 恢复之前忽略的文件
8. 若需要撤销 add 的某个文件:git reset 文件名

**若需要撤销指定的某次提交（已经提交到远程）：**
1. 先 git log 查看 commit ID
2. 执行 git reset –-soft <版本号> ，如 git reset --soft 4f5e9a90edeadcc45d85f43bd861a837fa7ce4c7 ，重置至指定版本的提交，达到撤销提交的目的
3. 执行 git push origin 分支名 –-force ，强制提交当前版本号。此时远程库回滚完成。
4. 随后可以再次使用 status 重新提交

若出现“fatal: Not a git repository: mmdetection/…/.git/modules/mmdetection“之类的命令，
网上说 git init 可以解决，但我这儿不行；解决办法是 cd mmdetection，把里面的.git 删除，即 rm -rf .git*

**git 本地创建分支 并推送到远程新分支：**
git checkout -b dbg_lichen_star
参考：https://blog.csdn.net/ljj_9/article/details/79386306

参考：
https://blog.csdn.net/zcw4237256/article/details/78542122
https://www.cnblogs.com/qqhfeng/p/13380210.html
https://blog.csdn.net/f0rd_/article/details/117434986

https://www.cnblogs.com/chaoxiZ/p/9714085.html

**二 拉取代码到本地, 且不覆盖本地已有代码**
1.git stash
2.git pull
3.git stash pop
4.git stash list
https://blog.csdn.net/weixin_40367126/article/details/104197540
https://zhuanlan.zhihu.com/p/403557624

**三 切换不同分支**
git checkout
(如果另一个分支有相同的文件, 可能会引起覆盖警告, 可以先 stash 起来)

**四 自己的分支修改之后merge到主分支**
git checkout master
git merge --no-ff 自己的分支名

参考:https://blog.csdn.net/default7/article/details/123425422

## reference h
[Git实用教程（三） | Git本地库操作（仓库初始化、提交修改） - 知乎](https://zhuanlan.zhihu.com/p/87680115)