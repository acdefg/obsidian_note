## 双系统安装
## ubuntu_installation
### write iso into USB disk to build a system disk
- iso mirror link: https://mirrors.tuna.tsinghua.edu.cn/#
	![image](https://user-images.githubusercontent.com/48377634/166099113-ba4c3834-4e28-4ba3-ac4a-86e46f14d7ce.png)
	![image](https://user-images.githubusercontent.com/48377634/166099131-a6621a96-2d60-42d8-969f-1e42739594eb.png)
- ultraiso download link: https://ultraiso.en.softonic.com/
选试用就行
1. 文件-> 打开->选择下载好的iso
2. 启动-> 写入硬盘映像
3. 格式化->写入
	![image](https://user-images.githubusercontent.com/48377634/166101218-67cabc3a-7503-4a50-9c05-da1a5f459c09.png)
	![image](https://user-images.githubusercontent.com/48377634/166101201-fedacbe4-20f5-4e9b-be35-d4769398d95a.png)
	![image](https://user-images.githubusercontent.com/48377634/166101243-b9170479-59dd-4173-bc4a-9c03d30dba19.png)
- Installation: reboot computer
	1. Press F2 to enter boot system (most computer use F2, you can look up on internet)
	2. Make the USB option to be the first boot option, then save
	3. Installation interface
		1. 我选的英文模式，可选择中文![](https://s2.loli.net/2022/05/01/viecr8l5E6wJQAd.png)
		2. 这张拍糊了，选择想要的语言即可![](https://s2.loli.net/2022/05/01/gvyAMEoPc8rSVzC.png)
		3. 不联网安的快一点，但是安完还是会更新的![](https://s2.loli.net/2022/05/01/a4cI5hZoRwU2GJ7.png)
		4. 事实上，我倾向于选择 Minimal installation + both other options, 有时候会卡住，等很长时间，点退出重来就行![](https://s2.loli.net/2022/05/01/HnqySjcBVvLmuxF.png)
		5. 网上大部分教程都选的第三项，自己分区，我之前按照教程来，结果用到一半提醒/boot 分区内存不够，没办法得重装，这次不想搞了就直接选择第一个了![](https://s2.loli.net/2022/05/01/PL78Aic2WUmpdCk.png)
		6. continue![](https://s2.loli.net/2022/05/01/mR5xl6pfdzYutrT.png)
		7. 漏了一张选地点的，往中国一点默认上海就行，这张图自己设置就行![](https://s2.loli.net/2022/05/01/ixDcg6n8bOaFrVP.png)
		8. 然后会提醒重启，有一个界面会告诉你拔掉 U 盘，按 enter 键继续，之后就会以上图设置的账户登陆

### reference link
1. https://zhuanlan.zhihu.com/p/355314438
2. https://zhuanlan.zhihu.com/p/407175785
3. https://blog.csdn.net/codeHonghu/article/details/111940656 - good

## arrangement steps
1. 修改软件源，在 software&updates..，修改命令行复制粘贴快捷键
2. 改系统配置
	```shell
	sudo gedit /etc/sudoers
	```
3. 安装软件
	[[ubuntu安装记录#1 Chinese input method]]
	```shell
	sudo apt-get install ibus-pinyin
	```
	
	```shell
	sudo apt install vim
	```
	
	[[ubuntu安装记录#3 Obsidian]]
	```shell
	sudo apt install xclip
	```
	
	```shell
	sudo apt install thunderbird
	```
	[[ubuntu安装记录#5 截图软件]]可直接配置
	```shell
	sudo apt install flameshot
	```
	[[ubuntu安装记录#6 Zsh 安装]]
	```shell
	sudo apt install curl
	```
	[[ubuntu安装记录#6 Zsh 安装]]
	```shell
	sudo apt-get install zsh
	```
	[[ubuntu安装记录#7 Git 安装]]可直接配置
	```shell
	sudo apt-get install git
	```
4. 下载百度网盘
[[ubuntu安装记录#2 百度网盘]]
下载 ubuntu 文件夹，配置 [[ubuntu安装记录#3 Obsidian]]  [[ubuntu安装记录#4 Picgo]]

### 1. Chinese input method
千万不要安装 fctix，装了重启直接卡死，我重装了两次才发现问题，直接安装 ibus-pinyin
```shell
sudo apt-get install ibus-pinyin
```

```shell
sudo apt install ibus-clutter
```
打开 language support，第一次安装会提示下载，同意即可
![300](https://s2.loli.net/2022/05/01/3sOh7xnjtCXRz6i.png)

重启，打开设置
![500](https://s2.loli.net/2022/05/01/n8J2FMjIXcwEP4t.png)
候选词变成 8 个
![](https://s2.loli.net/2022/05/01/X8x3rliuHYpRcqT.png)
切换快捷键在设置 Keyboard Shortcuts 里，图示即默认
![500](https://s2.loli.net/2022/05/01/73CFT1xJ865ocZH.png)
参考：[ubunut下安装ibus_pinyin中文输入法 - 学习那些事儿 - 博客园](https://www.cnblogs.com/yulongzhou/p/6345611.html#:~:text=%E5%AE%89%E8%A3%85ibus%20%E5%9C%A8%E8%BD%AF%E4%BB%B6%E4%B8%AD%E5%BF%83%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85%E5%8D%B3%E5%8F%AF%E6%88%96%E8%80%85sudo%20apt-get%20install%20ibus-pinyin,3%E3%80%81%E5%AE%89%E8%A3%85%E5%AE%8C%E4%B9%8B%E5%90%8E%E9%9C%80%E9%87%8D%E5%90%AF%E6%9C%BA%E5%99%A8%206%E3%80%81%E8%AE%BE%E7%BD%AE-%E6%96%87%E6%9C%AC%E8%BE%93%E5%85%A5-%E7%82%B9%E5%87%BB%E8%BE%93%E5%85%A5%E6%BA%90%E7%9A%84%E2%80%9C%2B%E2%80%9D%2C%E9%80%89%E6%8B%A9%E6%B1%89%E8%AF%AD%20%28Pinyin%29%20%28IBus%29%207%E3%80%81%E5%A6%82%E6%9E%9C%E8%BF%98%E6%B2%A1%E6%9C%89%E6%AD%A3%E5%B8%B8%E8%BE%93%E5%85%A5%E4%B8%AD%E6%96%87%EF%BC%8C%E5%B0%B1%E5%9C%A8%E5%B1%8F%E5%B9%95%E5%8F%B3%E4%B8%8A%E8%A7%92%E7%94%B5%E9%87%8F%E5%B7%A6%E8%BE%B9%E7%82%B9%E5%87%BB%E9%80%89%E6%8B%A9%E4%B8%80%E4%B8%8B%E5%B0%B1ok%E4%BA%86)
### 2. 百度网盘
[百度网盘 客户端下载](https://pan.baidu.com/download)
```
sudo dpkg -i ./baidunetdisk_4.3.0_amd64.deb
```
### 3. Obsidian
#### 插件
1. Setting (left botton)-> Community plugins -> turn off the safe mode
2. Copy all the backup documents and unzip all the plugins into ./obsidian with setting up a new folder named `plugins`
3. Setting (left botton)-> Community plugins->turn on all the plugins
#### 设置
1. About 里面可以改语言
2. 核心插件打开：大纲、日记等
3. 快捷键：
### 4. Picgo
[[图床设置]]
### 5. 截图软件
[[简单安装软件]]
![[obsidian安装与配置#图床的使用]]
### 6. Zsh 安装
[[zsh安装和设置]]
### 7. Git 安装
[[git操作]]
### 8. 美化
```shell
sudo apt install gnome-tweaks
```
![500](https://s2.loli.net/2022/05/06/gLBRxCMS87wXIm5.png)
   ![400](https://s2.loli.net/2022/05/06/tjlenkQ9KMoA73I.png)



#### Reference
[Ubuntu 20.04 系统美化--探索之旅（桌面环境，登录界面，开机动画，引导界面） - 知乎](https://zhuanlan.zhihu.com/p/401763253?utm_source=pocket_mylist)

## delete double system
### change grub option
1. boot interface
2. use easyUEFI
### delete the disk at disk manager
![image](https://user-images.githubusercontent.com/48377634/166102114-06da85b6-5330-4d1f-b32e-f5c926d13dc9.png)
把ubuntu几个分区都删掉，小心不要删错
### delete the option at EFI
1. 输入【Win】+【R】，输入【diskpart】打开diskpart；

2. 输入【list disk】，显示磁盘列表

3. 输入【select disk 0】，选择磁盘0，即win10系统所在磁盘；

4. 输入【list partition】，查看磁盘0的分区列表；

5. 输入【select partition 3】，选择wind10启动引导项所在分区（即Type=System，容量一般较小为100M的那一个分区）；

6. 为win10的EFI启动引导项所在分区分配盘符，输入【assign letter = p】，这里p为盘符名称，字母A~Z应该都可以，注意不要和已有盘符名重复即可；
7. 以管理员方式运行记事本，打开p盘，EFI文件夹，删除ubuntu文件夹

### reference link
https://www.cnblogs.com/arxive/p/11749770.html

