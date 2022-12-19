[01.套接字-字节序问题_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=127&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
讨论：跨主机传输要注意的问题
### 1 字节序问题
大端：低地址处放高字节
小端：低地址处放低字节
区分主机字节序 host 和网络字节序 network
解决：\_to\_  \_: htons,htonl,ntohs,ntohl
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
代码：
[06:00](https://www.bilibili.com/video/BV1yJ411S7r6?p=129&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=360.092404)


流式嵌套字