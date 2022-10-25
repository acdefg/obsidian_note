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
```  
=======
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
>>>>>>> 5ae9acaaeca9f50f245e655a8faf7e8ba113bbb6
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
git push -u origin main
git push -u origin main
```
`git push -u origin main` 改分支名字，可以不改，默认是master，第三句变为`git push -u origin master`

### obsidian使用git备份
安装obsidian_git插件，这个只是一个自动上下拉的插件，得先按前面的方法配置好远程仓库和本地仓库的链接。
还可以用gitee：git commit -m用于提交暂存区的文件，git commit -am用于提交跟踪过的文件。

### 删除操作
删除掉已经commit的文件，因为太大了一直导致push失败
参考链接：先看1，再看2
1. [Git清理commit中历史提交的大文件 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1536481)
2. [记一次删除Git记录中的大文件的过程-HollisChuang's Blog](https://www.hollischuang.com/archives/1708)

一些教程记录
[Fetching Title#15ws](https://blog.csdn.net/qq_34842671/article/details/70916587)  ---win下git安装教程
[Git: ‘LF will be replaced by CRLF the next time Git touches it‘ 问题解决与思考_Babylonxun的博客-CSDN博客](https://blog.csdn.net/Babylonxun/article/details/126598477)
[Fetching Title#ty7c](https://www.liaoxuefeng.com/wiki/896043488029600) --git教程  --廖雪峰

