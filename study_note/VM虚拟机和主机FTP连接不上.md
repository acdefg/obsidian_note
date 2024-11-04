使用 filezila 和 vsftpd

## 虚拟机
虚拟机机器安装 vsftpd 并且配置，下的虚拟机已经配置好了，但是没办法 ping 通主机
### vsftpd
这篇比较详细，主要就是下好去 conf 里面写对应设置，重启就好了
[Vsftp安装配置（超详细版）\_vsftpd-CSDN博客](https://blog.csdn.net/m0_64304713/article/details/133790019)
记录：
```
#查看vsftpd是否打开
netstat -lnpt | grep vsftpd 
#vsftpd配置文件夹 修改vsftpd.conf
cd /etc/vsftpd/
#重新启动vsftpd，修改完配置都需要
systemctl restart vsftpd
#查看地址 192开头
ifconfig
```

## windows
检查是否能连上：
```
ping 【上面ifconfig得到的地址】
```
### 防火墙
防火墙端口开放，在 vsftpd 配置文件中没有修改端口号，默认需要开启 20-22
网络和 Internet---> 高级网络设置 ---- > 防火墙和网络保护 ----> 高级设置  ----> 入站规则 ----> 新建规则 -----> 端口 ----> TCP（勾上） 特定端口：20-22
### VM
设置---->网络选 nat
### 还连不上
一套流程下来就行了，主要看网络配置，其他的自己定
[超详细虚拟机与主机网络连接以及互Ping不通问题的解决\_桥接模式windows2008虚拟机ping主机-CSDN博客](https://blog.csdn.net/weixin_41538012/article/details/115325944)
### filezila
ip 填 `ifconfig` 得到的 ip，用户和密码填配置好有权限的用户
