## 串口调试

> [!todo] Info
> 未完成：可能 sd 卡中 linux 系统文件有问题所有未能启动

下载安装 CP210x_Windows_Drivers.exe
驱动安装好以后，用红色 USB 线连接电脑 USB 口和开发板上的 UART 口进行连接, 然后打开电脑的设备管理器，设备管理器能够找到串口设备 CP210x, 找到映射端口，图为 COM3。如果不能成功安装驱动，可以尝试使用驱动精灵安装
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410282259633.png)
打开设备管理器：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410282259684.png)
使用 putty 或者其他串口调试工具连接
Serial line 填写 COM3，Speed 填写 115200，COM3 串口号根据设备管理器里显示的填写，选择 Serial，并将流控 Flow control 改为 None，点击“Open”
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410282303403.png)
打开开发板上的电源开关，PuTTY 工具窗口会显示 u-boot 和 Linux 系统的启动信息，（启动过程中不要输入命令，不然系统会停在 Uboot 阶段）

5. Linux系统启动完成后，可以在串口终端登陆系统，用户: root，密码: root
（有很多人是第一次接触Putty，或者是第一次用串口，需要说明的是，Putty输入命令是通过主机键盘输入，不是通过连接在开发板上的键盘输入）