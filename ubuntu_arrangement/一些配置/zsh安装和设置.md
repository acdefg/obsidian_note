## 安装
1. 安装zsh：
	```shell
	sudo apt-get install zsh
	```

2. 查看系统可用shell：
	```shell
	cat /etc/shells
	```
	![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220419175316.png)

1. 更换默认shell为zsh：
	```shell
	sudo chsh -s /bin/zsh
	```
	切换到root用户就不用sudo，这个设置后要登出重新登陆，再次打开就可以进入配置引导界面
	确认系统默认shell：
	```shell
	echo $SHELL
	```
	成功回复/bin/zsh
	![](https://s2.loli.net/2022/05/01/6QC3pFh5AHnWzbj.png)
	- q: 退出，下次打开还会显示配置界面
	- 0: 退出，并且生成空白.zshrc文件
	- 1: 进入设置
		-   `1`：设置命令历史记录相关的选项
		-   `2`：设置命令补全系统
		-   `3`：设置热建
		-   `4`：选择各种常见的选项，只需要选择“On”或者“Off”
		-   `0`：退出，并使用空白（默认）配置
		-   `a`：终止设置并退出
		-   `q`：退出
	建议选择 0，因为后面会配置.zshrc
2. 上面那个可能不成功可以尝试：
	```shell
	~$ sudo chsh  
	[sudo] password for username:  
	正在更改 root 的 shell  
	请输入新值，或直接敲回车键以使用默认值  
    登录 Shell [/bin/tcsh]: /bin/zsh
	```
	然后直接输入 `zsh` 就可以得到上面的界面

3. 安装 oh-my-zsh
	[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)make set zsh more easy
	installation code:
	**curl**：要使用 curl 要先安装 `sudo apt install curl`
	
	```shell
	sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh )"
	``` 
	
	**wget**
	
	`sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
	
	**fetch**
	
	`sh -c "$(fetch -o - https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
	after successfully installing:
	![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220419175238.png)
	it needs some time
## 配置文件.zshrc 
### 别名
my:
```zsh
 alias zshconfig="vim ~/.zshrc"
 alias szsh="source ~/.zshrc" 
 alias ohmyzsh="cd ~/.oh-my-zsh"
 alias clean="sudo apt clean"
 alias remove="sudo apt autoremove"
 alias update="sudo apt update"
 alias upgrade="sudo apt upgrade"
```
>这个原来zsh和oh my zsh 第一个为mate，还没搞清楚用法

reference:
```zsh
alias cls='clear'
alias ll='ls -l'
alias la='ls -a'
alias vi='vim'
alias grep="grep --color=auto"
```
### 主题
- 随机主题
```txt
# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
```
这一段说的很清楚了，其实不用echo，每次打开他都会显示载入了哪个主题
zsh 5.8 (x86_64-ubuntu-linux-gnu)
- 指定主题
```zsh
 ZSH_THEME= "random" 
```
修改random为任意主题名即可

### 插件
- 第一种：
```shell
sudo apt-get install zsh-autosuggestions zsh-syntax-highlighting zsh-theme-powerlevel9k
```
theme插件可选，其余两个推荐，一个是命令自动建议，一个是语法高亮，正确为绿色
键入`vim ~/.zshrc`，最后加入
```
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
修改plugins部分如下
```
 plugins=(
   git
   z
 )  
```
开启新的 Shell 或执行 `souce ~/.zshrc`，就可以开始体验插件了
z 是一个文件夹快捷跳转插件，对于曾经跳转过的目录，只需要输入最终目标文件夹名称，就可以快速跳转，避免再输入长串路径，提高切换文件夹的效率。
- 第二种
1.  把插件下载到本地的 `~/.oh-my-zsh/custom/plugins` 目录：

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

2. 在 `.zshrc` 中，把 `zsh-autosuggestions` 加入插件列表：

```bash
plugins=(
    # other plugins...
    zsh-autosuggestions  # 插件之间使用空格隔开
)
```

3. 开启新的 Shell 或执行 `souce ~/.zshrc`，就可以开始体验插件。

zsh-syntax-highlighting is the same
```text
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting 
```

```text
plugins=(
    # other plugins...
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```

## 好看但没用的东西
终端黑客帝国屏保

```bash
# 安装
sudo apt install cmatrix

# 运行（加上 -lba 参数看起来更像电影，加上 -ol 参数起来更像 Win/Mac 的屏保）
cmatrix
```


## 参考
1. https://zhuanlan.zhihu.com/p/139305626?utm_source=pocket_mylist
2. [技术|配置一个简洁高效的 Zsh](https://linux.cn/article-13030-1.html)