## 重装系统
将原来的 ubuntu18.04 换成 raspberry OS 64 bit full
在安装时设置了用户名、密码和无线 wifi，通过改配置文件的方式连接 wifi 失败，但是在安装时直接设置会自动导入电脑所连 wifi，可以成功。

在 ubuntu 主机上安装 Putty：
```shell
sudo apt-get install putty
```
获取 IP 地址：
192.168.31.84
SSH 连接时，之前这个 IP 连接过了，要删一下原来的文件才能用：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211101930369.png)
在树莓派上运行 raspi-config，启用 VNC：
[Putty保持会话连接 & 正确注销方法_Tartisan的博客-CSDN博客](https://blog.csdn.net/Design_by_TaoZ/article/details/80629646)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211101944468.png)
VNC 连接：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211101944590.png)

一开始不能显示桌面，重新连接树莓派，进入config改了分辨率之后可以了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211101953207.png)

## 内核编译方法
三种实现方法（供组员参考不用放到报告里）：
1. 直接在树莓派中编译
2. 在 linux 上编译好，放到 sd 里
	1. Make config
	2. Make menuconfig
3. Sd 卡挂载到 linux 上，不取 SD 卡更新内核（待定）
[树莓派不取 SD 卡更新 kernel 和 dtb_Li-Yongjun的博客-CSDN博客](https://blog.csdn.net/lyndon_li/article/details/127718815)

使用 uname -a 察看内核版本
查看原来内核版本
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110125692.png)
查看下载内核版本
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110125244.png)
复制.Config 文件
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110126615.png)
编译内核：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110126065.png)

在 linux 上编译好，放到 sd 里：

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110100666.png)
输入命令 " dmesg "看看 SD 卡是否挂在成功
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110101349.png)
第一次挂载错了地方，取消掉
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110111889.png)
Lsblk: 重新查看挂载对不对
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110119639.png)
Install modules：安装模块
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110122475.png)

替换内核：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110133965.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110956391.png)

Menuconfig：修改配置文件
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211111027509.png)


### 第二次安装
这一部分仅提供内核升级对比的图片，是第一次安装不对后重新尝试的，内核版本的区别是安装系统的内核版本和从 github 上下载的内核版本之间的区别
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211110935973.png)
内核升级：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211111006632.png)

## 内核配置
这里自己修改内核
### 几种方法
常见的几种配置方式：（仅供参考）

为了完成内核的配置，必须切换到root用户，然后转入内核源码目录(就是你下载新内核的目录)：

#cd /usr/src/linux/linux-2.6.38

然后执行下面命令之一:

#make config
#make oldconfig
#make menuconfig


1.make config

基于文本的最为传统的也是最为枯草的一种配置方式，但是它可以使用任何情况，这种方式会为每一个内核支持的特性向用户提问，如果用户回答“y”，则把特性编译进内核；回答“m”，则它特性作为模块进行编译；回答“n”，则表示不对该特性提供支持

如果回答每个问题前，必须考虑清楚，如果在配置过程中犯了错误给了错误的回答，就只能按“ctcl+c”强行退出了

2.make oldconfig

make oldconfig和make config类似，但是它的作用是在现有的内核设置文件基础上建立一个新的设置文件，只会向用户提供有关新内核特性的问题，在新内核升级的过程 中，make oldconfig非常有用，用户将现有的配置文件.config复制到新内核的源码中，执行make oldconfig，此时，用户只需要回答那些针对新增特性的问题

make silentoldconfig : Like above, but avoids cluttering the screen with questions already answered.和上面oldconfig一样，但在屏幕上不再出现已在.config中配置好的选项。

3.make menuconfig

基于终端的一种配置方式，提供了文本模式的图形用户界面，用户可以通过光标移动来浏览所支持的各种特性。使用这用配置方式时，系统中必须安装有ncurese库，否则会显示“Unable to find the Ncurses libraies”的错误提示

4.make xoncifg

基 于X Winodws的一种配置方式，提供了漂亮的配置窗口，不过只有能够在X Server上使用root用户欲行X应用程序时，才能够使用，它依赖于QT，如果系统中没有安装QT库，则会出现“Unable to find the QT installation”的错误提示


### Start
Backup
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211111306492.png)
Menuconfig：配了几个选项还没试
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211111027509.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211121857732.png)

## Reference
[树莓派内核源码得获取，配置， 编译，裁剪_一只青木呀的博客-CSDN博客_树莓派内核源码](https://blog.csdn.net/weixin_45309916/article/details/107525503) 👍官网搬运工
[树莓派 raspi-config 设置详解_weixin_34150830的博客-CSDN博客](https://blog.csdn.net/weixin_34150830/article/details/91733122)   --read
[【嵌入式】构建嵌入式Linux系统（uboot、内核、文件系统）_萌宅鹿同学的博客-CSDN博客_构建嵌入式系统](https://blog.csdn.net/weixin_43734095/article/details/105251245)
[Linux内核配置选项 （经典学习）_wangliang888888的博客-CSDN博客](https://blog.csdn.net/wangliang888888/article/details/86599092) 👍参考选项配置
[技术|如何装载/卸载 Linux 内核模块](https://linux.cn/article-9750-1.html) 有关于查看模块数量的说明
[【嵌入式】构建嵌入式Linux系统（uboot、内核、文件系统） - 知乎](https://zhuanlan.zhihu.com/p/573207792) 介绍内核系统
[嵌入式Linux内核制作_EmbededCoder的博客-CSDN博客](https://blog.csdn.net/u012308586/article/details/89491295) linux 内核裁剪流程
[Linux内核架构及内核裁剪 - 知乎](https://zhuanlan.zhihu.com/p/359566401?utm_campaign=shareopn&utm_medium=social&utm_oi=1192924132751323136&utm_psn=1574350599541170177&utm_source=wechat_session&utm_id=0)  流程参考
[嵌入式Linux内核制作_EmbededCoder的博客-CSDN博客](https://blog.csdn.net/u012308586/article/details/89491295) make clean 等（待定）
