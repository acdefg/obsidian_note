[01.套接字-字节序问题_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yJ411S7r6?p=127&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
讨论：跨主机传输要注意的问题
### 1 字节序问题
大端：低地址处放高字节
小端：低地址处放低字节
区分主机字节序 host 和网络字节序 network
\_to\_  \_: htons,htonl,ntohs,ntohl
### 2 对齐
struct{
	int i;
	float f;
	char ch;
}

报式嵌套字
流式嵌套字