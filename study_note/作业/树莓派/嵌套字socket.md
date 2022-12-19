[01.套接字-字节序问题_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=127&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
讨论：跨主机传输要注意的问题
### 1 字节序问题
大端：低地址处放高字节
小端：低地址处放低字节
区分主机字节序 host 和网络字节序 network
解决：\_to\_  \_: htons,htonl,ntohs,ntohl
>解释如下，数字 16 的 16 进制表示为 0x0010，数字 4096 的 16 进制表示为 0x1000。 由于 Intel 机器是小尾端，存储数字 16 时实际顺序为 1000，存储 4096 时实际顺序为 0010。因此在发送网络包时为了报文中数据为 0010，需要经过 htons 进行字节转换。如果用 IBM 等大尾端机器，则没有这种字节顺序转换，但为了程序的可移植性，也最好用这个函数。
原文链接：https://blog.csdn.net/zouxinfox/article/details/1814088
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

流式嵌套字