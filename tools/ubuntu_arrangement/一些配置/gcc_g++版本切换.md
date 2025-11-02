[gcc/g++多版本切换 (ubuntu18.04) - Angry\_Panda - 博客园](https://www.cnblogs.com/devilmaycry812839668/p/10351763.html)
这篇帖子很全面

查看已安装的 gcc，g++

```shell
sudo updatedb && sudo ldconfig
locate gcc | grep -E "/usr/bin/gcc-"
#如果locate不能用
ls /usr/bin/gcc*
ls /usr/bin/g++*
```

查看目前使用版本
```shell
g++ --version
g++ -v
```

查看可替换版本
```shell
#显示
sudo update-alternatives --display gcc
#配置
sudo update-alternatives --config gcc
```

可替换版本安装以及优先级设定
```shell
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11
```
系统会自动选择优先级最高的版本

另外，还有一种方法，可以修改默认的 g++版本，我们可以更改一下 gcc 的软链接:

```
sudo rm /usr/bin/gcc  
sudo ln -s /usr/bin/gcc-4.8 /usr/bin/gcc  
sudo rm /usr/bin/g++  
sudo ln -s /usr/bin/g++-4.8 /usr/bin/g++
```

### bug
之前安装了 cuda，导致上述方法更改 gcc 和 g++不版本均无效
查到 which 在指向的 gcc 了路径是
```
which g++                                       
/usr/local/cuda/bin/g++
```
改不掉这个路径
再看一下这个路径的指向
```
ls -al /usr/local/cuda/bin/g++*                 
lrwxrwxrwx 1 root root 12  8月 23 23:33 /usr/local/cuda/bin/g++ -> /usr/bin/g++-10
```
原来就是这个东西，导致我一直卡在版本 10
```
#删掉/usr/local/cuda/bin/g++，重新定向
sudo rm /usr/local/cuda/bin/g++                                        1 ↵
sudo ln -s /usr/bin/g++  /usr/local/cuda/bin/g++
```
成功了，折磨我一晚上，生气！