参考：[记一次Ubuntu完美迁移系统盘的折腾_记一次完美ubuntu 迁移_Bruski的博客-CSDN博客](https://blog.csdn.net/Bruski/article/details/115840667)
### 主要步骤
1. 制作原系统的启动盘：因为原来的系统已经满了，完全开不了机了，所以是用 windows 安装的，参考 [[ubuntu安装记录]]
2. 从启动盘启动：这里我试过第一个选项 try ubuntu 将进入但是显示高亮度屏幕，所以每次进去都是选择第二个 safe mode，然后进入之后再选择 try ubuntu
3. 处理分区：参考链接中 ubuntu 个根据功能划分了好几个分区，当时偷懒装系统的时候就选择的自动分区，于是就只有一个盘，所以只需要将新的盘格式化成 ext4，和原来的一样即可，然后第一次忘记选上面的勾，就以为已经开始了
4. 复制文件：从这步开始可以进入 root，几乎所有指令都需要 root 其权限，复制代码如下，具体根据盘符来修改
```shell
dd if=/dev/nvme0n1p5 of=/dev/nvme1n1p2
```
想要看进度，新开一个 terminal 
```shell
watch -n 5 killall -USR1 dd
```
原来的 terminal 将就会输出进度
5. 修改 uuid：先生称再修改老是失败，然后两条一起输成功了
```shell
uuidgen | xargs tune2fs /dev/nvme1n1p2 -U
```
查看 uuid：
```shell
sudo blkid
```
6. 修复启动项：联网，下载一个 boot-repair，手动修复太麻烦了，直接recommand
```shell
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt install -y boot-repair
boot-repair
```

### 问题
```ad-failure

弄完之后，新系统的启动项有了，原来盘的启动项也还在，然后直接把原来盘给删了，结果系统启动出错了，不过问题不大，如下图
```

### 原博客里有两步没有做
可能是这两步导致的问题
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202303170949790.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202303170949340.png)

