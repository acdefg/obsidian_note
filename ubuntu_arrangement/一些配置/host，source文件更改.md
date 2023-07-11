# host 文件更改
``` shell
sudo gedit /etc/hosts
sudo vim /etc/hosts
```
## github
https://github.com/ineo6/hosts - 复制里面的内容即可（会更新）
Fastgithub 可以自动改，没装，参考：https://zhuanlan.zhihu.com/p/428454772

# 软件源 source 更改
参考链接：[Ubuntu20.04软件源更换 - 知乎](https://zhuanlan.zhihu.com/p/142014944)
复制的内容在：[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)
简要步骤：
1.备份原来的源，将以前的源备份一下，以防以后可以用的。

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2.打开/etc/apt/sources.list文件，在前面添加如下条目，并保存。

```shell
sudo vim /etc/apt/sources.list
```

（可将 vim 更换为自己熟悉的编辑器）

3.更新

更新源

```text
sudo apt-get update
```

如出现依赖问题，解决方式如下：

```text
sudo apt-get -f install
```

更新软件：

```text
sudo apt-get upgrade
```

## 各种镜像源的地址
附上网上找的各种源的地址：

1.企业源：  
阿里云开源镜像站： [http://mirrors.aliyun.com/](https://link.zhihu.com/?target=http%3A//mirrors.aliyun.com/)  
搜狐开源镜像站：[http://mirrors.sohu.com/](https://link.zhihu.com/?target=http%3A//mirrors.sohu.com/)  
网易开源镜像站：[http://mirrors.163.com/](https://link.zhihu.com/?target=http%3A//mirrors.163.com/)

2.教育源：  
重庆大学：  
[http://mirrors.cqu.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.cqu.edu.cn/)  
北京理工大学：  
[http://mirror.bit.edu.cn](https://link.zhihu.com/?target=http%3A//mirror.bit.edu.cn/) (IPv4 only)  
[http://mirror.bit6.edu.cn](https://link.zhihu.com/?target=http%3A//mirror.bit6.edu.cn/) (IPv6 only)  
北京交通大学：  
[http://mirror.bjtu.edu.cn](https://link.zhihu.com/?target=http%3A//mirror.bjtu.edu.cn/) (IPv4 only)  
[http://mirror6.bjtu.edu.cn](https://link.zhihu.com/?target=http%3A//mirror6.bjtu.edu.cn/) (IPv6 only)  
[http://debian.bjtu.edu.cn](https://link.zhihu.com/?target=http%3A//debian.bjtu.edu.cn/) (IPv4+IPv6)  
兰州大学：[http://mirror.lzu.edu.cn/](https://link.zhihu.com/?target=http%3A//mirror.lzu.edu.cn/)  
厦门大学：[http://mirrors.xmu.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.xmu.edu.cn/)  
上海交通大学：  
[http://ftp.sjtu.edu.cn/](https://link.zhihu.com/?target=http%3A//ftp.sjtu.edu.cn/) (IPv4 only)  
[http://ftp6.sjtu.edu.cn](https://link.zhihu.com/?target=http%3A//ftp6.sjtu.edu.cn/) (IPv6 only)  
清华大学：  
[http://mirrors.tuna.tsinghua.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.tuna.tsinghua.edu.cn/) (IPv4+IPv6)  
[http://mirrors.6.tuna.tsinghua.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.6.tuna.tsinghua.edu.cn/) (IPv6 only)  
[http://mirrors.4.tuna.tsinghua.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.4.tuna.tsinghua.edu.cn/) (IPv4 only)  
天津大学：[http://mirror.tju.edu.cn/](https://link.zhihu.com/?target=http%3A//mirror.tju.edu.cn/)  
中国科学技术大学：  
[http://mirrors.ustc.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.ustc.edu.cn/) (IPv4+IPv6)  
[http://mirrors4.ustc.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors4.ustc.edu.cn/)  
[http://mirrors6.ustc.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors6.ustc.edu.cn/)  
西南大学：[http://linux.swu.edu.cn/swudownload/Distributions/](https://link.zhihu.com/?target=http%3A//linux.swu.edu.cn/swudownload/Distributions/)  
东北大学：  
[http://mirror.neu.edu.cn/](https://link.zhihu.com/?target=http%3A//mirror.neu.edu.cn/) (IPv4 only)  
[http://mirror.neu6.edu.cn/](https://link.zhihu.com/?target=http%3A//mirror.neu6.edu.cn/) (IPv6 only)  
电子科技大学：[http://ubuntu.uestc.edu.cn/](https://link.zhihu.com/?target=http%3A//ubuntu.uestc.edu.cn/)  
青岛大学：[http://mirror.qdu.edu.cn/](https://link.zhihu.com/?target=http%3A//mirror.qdu.edu.cn/)  
开源中国社区 [http://mirrors.oss.org.cn/](https://link.zhihu.com/?target=http%3A//mirrors.oss.org.cn/)  
大连东软信息学院 [http://mirrors.neusoft.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.neusoft.edu.cn/)  
华中科技大学 [http://mirrors.hust.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.hust.edu.cn/)  
中山大学 [http://mirrors.sysu.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.sysu.edu.cn/)  
清华大学学生网管会 [http://mirrors.tuna.tsinghua.edu.cn/](https://link.zhihu.com/?target=http%3A//mirrors.tuna.tsinghua.edu.cn/)  
浙江大学 [http://mirrors.zju.edu.cn/web/](https://link.zhihu.com/?target=http%3A//mirrors.zju.edu.cn/web/)

台湾淡江大学 [http://ftp.tku.edu.tw/Linux/](https://link.zhihu.com/?target=http%3A//ftp.tku.edu.tw/Linux/)

Linux运维派开源镜像 [http://mirrors.skyshe.cn/](https://link.zhihu.com/?target=http%3A//mirrors.skyshe.cn/)

各种源原始地址：[中国Linux源镜像站大全 - starnight_cyber - 博客园](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/Hi-blog/p/5954230.html)