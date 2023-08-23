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
```

查看可替换版本
```shell
sudo update-alternatives --config gcc
```

可替换版本安装以及优先级设定
```shell
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11
```
系统会自动选择优先级最高的版本