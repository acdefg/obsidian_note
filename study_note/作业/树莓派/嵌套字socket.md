[01.套接字-字节序问题_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=127&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
讨论：跨主机传输要注意的问题
### 1 字节序问题
大端：低地址处放高字节
小端：低地址处放低字节
区分主机字节序 host 和网络字节序 network
解决：\_to\_  \_: htons,htonl,ntohs,ntohl
```ad-note

解释如下，数字 16 的 16 进制表示为 0x0010，数字 4096 的 16 进制表示为 0x1000。 由于 Intel 机器是小尾端，存储数字 16 时实际顺序为 1000，存储 4096 时实际顺序为 0010。因此在发送网络包时为了报文中数据为 0010，需要经过 htons 进行字节转换。如果用 IBM 等大尾端机器，则没有这种字节顺序转换，但为了程序的可移植性，也最好用这个函数。
原文链接：https://blog.csdn.net/zouxinfox/article/details/1814088
```

### 2 对齐
struct{
	int i;
	float f;
	char ch;
}
[20:08](https://www.bilibili.com/video/BV1yJ411S7r6?p=127&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1208.327715)
当前地址能否整除 sizeof
解决：不对齐
### 3 类型长度问题
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219120558.png)
## socket
[02.套接字-socket函数_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=128&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219121243.png)
#### socket 函数
##### 协议族 domain
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219121552.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219121552.png)
标记的是内核和用户态的一个通信

##### 传输类型
[18:15](https://www.bilibili.com/video/BV1yJ411S7r6?p=128&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1095.711557)
流式：有序可靠双工字节传输
报式：数据分组传输、无连接的、不可靠的、最大上限是固定

##### protool
domain 中的一个

##### 返回文件苗符

#### 报式嵌套字
[03.套接字-报式套接字相应过程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
套路 1：
[02:37](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=157.780979)
![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219143503.png)
##### 代码
[06:00](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=360.092404)

[10:19](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=619.293297)
1024 以上的端口
被动方：
[11:57](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=717.070315)

[13:00](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=780.55317)
![100](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219144212.png)
AF_**INET**（又称 PF_**INET**）是 IPv4 网络协议的套接字类型
0：默认
![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219144437.png)
bind：
[15:13](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=913.213163)
man 7 ip：
[17:37](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1057.120534)
code：
[19:30](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1170.394879)
sin_addr：点分式的内容转化为大整数
[22:58](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1378.023747)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219145842.png)
0.0.0.0：any address
[25:18](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1518.675878)
[04.套接字-报式套接字实例_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
bind code：
[00:39](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=39.691269)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212192112463.png)

recvfrom 报式，recv 流式
[02:00](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=120.641185)
code：
[04:44](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=284.719147)
inet_ntop：int to p
[09:27](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=567.574093)
code:
[12:53](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=773.136803)

[13:39](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=819.703748)

[14:05](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=845.038481)
netstat -anu
[14:36](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=876.376754)

sender:
[16:05](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=965.171723)

[16:44](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1004.051567)
![100](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221219152130.png)

send to:
[17:53](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1073.096463)
code:
[21:07](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1267.65127)
[22:05](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1325.624664)

[24:00](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1440.404608)
[25:29](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1529.85408)

bug
[25:50](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1550.2303)
[27:44](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1664.766653)

[29:41](https://www.bilibili.com/video/BV1yJ411S7r6?p=130&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1781.836432)

[05.套接字-动态报式套接字实例_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=131&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
变长数组
[03:20](https://www.bilibili.com/video/BV1yJ411S7r6?p=131&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=200.898168)
sender
[05:57](https://www.bilibili.com/video/BV1yJ411S7r6?p=131&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=357.250988)
[15:18](https://www.bilibili.com/video/BV1yJ411S7r6?p=131&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=918.642441)
receiver
[11:34](https://www.bilibili.com/video/BV1yJ411S7r6?p=131&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=694.041279)

[06.套接字-多播实例1_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
多点通讯：广播（全网广播，子网广播），多播/组播
广播：
code:
[05:07](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=307.288932)
man 7 socket
[07:14](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=434.375716)
socket option

setsockop,getsockop:
[12:02](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=722.006624)
code:
[14:55](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=895.215316)
[18:02](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1082.142452)

phenomeno
[20:49](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1249.202385)

receiver code:
[22:23](https://www.bilibili.com/video/BV1yJ411S7r6?p=132&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1343.029424)

[07.套接字-多播实例2_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=133)
防火墙
多播：
[08:04](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=484.031759)
code
[09:18](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=558.2049)
setsocketop code
[12:38](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=758.469491)
ifconfig，ip ad sh, if_nametoindex
[16:12](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=972.605186)
bugs
[17:50](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=1070.620953)
receiver:
[18:44](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=1124.872265)
phenomeno
[21:45](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=1305.021726)
多播组：
[23:04](https://www.bilibili.com/video/BV1yJ411S7r6?p=133#t=1384.320773)

流式嵌套字
[12.流式套接字详解_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=138&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212192046411.png)

[04:28](https://www.bilibili.com/video/BV1yJ411S7r6?p=138&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=268.207775)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212192053043.png)

[12:15](https://www.bilibili.com/video/BV1yJ411S7r6?p=138&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=735.945682)
listen：
[16:04](https://www.bilibili.com/video/BV1yJ411S7r6?p=138&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=964.926669)
accept:
[13.流式套接字实现实例_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=139&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212192111917.png)

nc 名模拟 client
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212192219235.png)
[18:43](https://www.bilibili.com/video/BV1yJ411S7r6?p=139&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1123.773251)
地址还在使用：
netstate -ant
[19:43](https://www.bilibili.com/video/BV1yJ411S7r6?p=139&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=1183.12822)
过段时间会自动回收
没有正常释放的情况下，想继续用 reuseaddr

client.c
[14.流式套接字并发实例_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=140&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
unix 一切皆文件

[15.流式套接字实现图片页面抓包_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=141&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
### reference link
[两个及其简单的TCPUDP程序，树莓派与pc间的通信_怪我不能1v9的博客-CSDN博客_实现笔记本和树莓派之间的tcp网络通信](https://blog.csdn.net/qq_40993036/article/details/111810975?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-111810975-blog-105521375.pc_relevant_3mothn_strategy_and_data_recovery&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-111810975-blog-105521375.pc_relevant_3mothn_strategy_and_data_recovery&utm_relevant_index=1)
[15.流式套接字实现图片页面抓包_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=141&spm_id_from=pageDriver&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
[树莓派和上位机使用TCP通信（字符串和图像传输）_嵌入式-小王的博客-CSDN博客_树莓派tcp通信](https://blog.csdn.net/Wangguang_/article/details/111658412)
[网络编程之UDP协议下C#编写服务端 树莓派作客户端实现发送接受消息_clyrjj的博客-CSDN博客](https://blog.csdn.net/clyrjj/article/details/109408261)