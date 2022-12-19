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

报式嵌套字
[02.套接字-socket函数_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=128&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)

流式嵌套字