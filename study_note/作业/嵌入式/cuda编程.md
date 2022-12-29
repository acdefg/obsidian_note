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
选了其中的一个 key，添加一下，会有 warning，正确解决如链接：[apt key - Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead - Stack Overflow](https://is.gd/hoVl1w) 事实上，不用管 warning，直接再 update 阿安装就可以了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292234809.png)
按顺序安装下面两个
```zsh
sudo apt-get install liburcu6 
sudo apt-get -y install cuda 
```
至此安装完成，查看一下下
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292241297.png)
添加环境变量，这条我改过
```zsh
zshconfig
```
添加这两句
```txt
export  PATH=/usr/local/cuda/bin:$PATH  
export  LD_LIBRARY_PATH=/usr/local/cuda/lib64$LD_LIBRARY_PATH
```
输入 `nvcc -V`，查看更改成功
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292244738.png)
