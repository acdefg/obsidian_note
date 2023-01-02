## cuda 安装
参考：[Ubuntu 22.04 安装cuda，适用20.04_AIhub的博客-CSDN博客_ubuntu22.04安装cuda](https://is.gd/H3L2qQ)
[kali linux 安装CUDA 11.6问题总结 - FreeBuf网络安全行业门户](https://www.freebuf.com/sectool/328870.html)
1. 安装驱动
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291206968.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291221320.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291452085.png)
2. 安装 cuda toolkit，参考[kali linux 安装CUDA 11.6问题总结 - FreeBuf网络安全行业门户](https://www.freebuf.com/sectool/328870.html)
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
## cuda 程序
cuda 编程教学👍：[CUDA C/C++ 教程一：加速应用程序_白水baishui的博客-CSDN博客_c++ cuda](https://is.gd/XcIHdt)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022103477.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022222652.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022316707.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022319208.png)

## 问题
上次之后不知道系统删掉了什么环境，这次运行就一直报错，找不到 opencv2/opencv.hpp 从查找文件还在，经过一步一步检查，应该是 autoremove 把 libopencv-dev 在这个包给删了，'sudo apt-get install libopencv-dev' ，重新安装，配置好 vscode，F5 运行没问题了。

[vs各个版本编写代码时的光标变成了黑块，黑块选中字符，再输入的时候就会替换掉那个黑块选中的字符_CJack酒杯的博客-CSDN博客_c语言光标变成黑块](https://blog.csdn.net/qwe6620692/article/details/88079003)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212301822118.png)
根据这篇把 gcc/g++降级了，还是不行 [error: parameter packs not expanded with ‘...’ · Issue #119 · NVlabs/instant-ngp · GitHub](https://github.com/NVlabs/instant-ngp/issues/119)
降级新办法：[[ubuntu][原创]ubuntu gcc g++降级方法_FL1623863129 的博客-CSDN 博客_ubuntu 22 gcc12 降到 11](https://blog.csdn.net/FL1623863129/article/details/115192387)

```shell
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 48
```

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212301823753.png)

