---
title: ubuntu code lines
tags: []
created: 星期六, 十一月 29日 2025, 11:18:00 晚上
modified: 星期日, 六月 28日 2026, 5:16:55 下午
---

# 1. 更新系统

```bash
# 更新本地包数据库
sudo apt update

# 更新所有已安装的包（也可以使用 full-upgrade）
sudo apt upgrade

# 自动移除不需要的包
sudo apt autoremove
```

这里补充几个常用的清理命令：

- apt autoclean: 将已删除软件包的.deb 安装文件从硬盘中删除；
- apt clean: 同上，但会把已安装的软件包的安装包也删除掉；
- apt autoremove: 删除为了满足其他软件包的依赖而安装，但现在不再需要的软件包；
- apt remove [软件包名]: 删除已安装的软件包（保留配置文件）；
- apt --purge remove [软件包名]: 删除已安装包（不保留配置文件）。

# VIM
# 复制、粘贴等

y: 复制在可视模式下选中的文本。  
yy or Y: 复制整行文本。
p: 在光标之后粘贴。  
P: 在光标之前粘贴。

# 插入
i: 在光标前插入；

> 一个小技巧：按 8，再按 i，进入插入模式，输入=， 按 esc 进入命令模式，就会出现 8 个=。 这在插入分割线时非常有用，如 30i+ 就插入了 36 个 + 组成的分割线。  

I: 在当前行第一个非空字符前插入；
a: 在光标后插入；  
A: 在当前行最后插入；  
o: 在下面新建一行插入；  
O: 在上面新建一行插入；

# 参考
1. https://blog.csdn.net/ch_improve/article/details/88706714 - vim 操作大全
2. Here are some games to help you master some basic operations in `vim`. Have fun!
	- [Vim Adventures](http://vim-adventures.com)
	- [Vim Snake](http://www.vimsnake.com)
	- [Open Vim Tutorials](http://www.openvim.com/tutorial.html)
	- [Vim Genius](http://www.vimgenius.com)
# Terminal
1. linux 操作# [Linux入门教程](https://ysyx.oscc.cc/docs/ics-pa/linux.html#linux入门教程) (ysyx)