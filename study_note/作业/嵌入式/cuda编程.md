## cuda 安装
1. 安装驱动
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291206968.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291221320.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291452085.png)
问题 1：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292226234.png)

```
sudo vim /etc/apt/sources.list
```
加上
```
deb http://ftp.de.debian.org/debian bullseye main
```
update 一下
问题 2：没有 key
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292233359.png)
选了其中的一个 key，添加一下，会有 warning，正确解决如链接：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292234809.png)
