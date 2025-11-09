以 obsidian 的安装为例：

## 运行
为下载好的 AppImage文件赋予可执行权限：
- 右键——>属性——>权限——>勾选下方“允许文件作为程序执行”
	或者
	在该文件路径下，终端输入 `chmod +x Obsidian-0.14.2.AppImage`
- 双击该文件就可以运行了

## 快捷方式添加
### 简单说明

需要切换到root模式

(possibly need root right, I've already changed root setting, see [[配置中一些小问题]])
```shell
cd /usr/share/applications

sudo vim obsidian.desktop
```
在该文件中写入：（需要改：Exec=(AppImage路径)，Icon=(logo路径)）
```shell
[Desktop Entry]
Name=Obsidian
Exec=/home/Obsidian/Obsidian-0.14.2.AppImage
Icon=/home/Obsidian/obsidian.png           
Type=Application
StartupNotify=true
```
或：
```shell
sudo -s
nautilus
```
手动找到/usr/share/applications
`sudo vim obsidian.desktop`加入一样的内容

### 找 ICON 的方法
实际上，我添加的desktop的内容是从appimage中解压出来的
参考：[AppImage 设置为图标启动(以 Wiznote和Navicat 为例)](https://blog.csdn.net/jiang_huixin/article/details/106037973) 包含两种APPIMAGE解压方式
在appimage文件路径中打开终端，输入
```shell
./Obsidian.AppImage --appimage-extract
```
进行解压，解压文件中包含.desktop和logo
.desktop 内容如下，可以直接用下面的，改变 Exec 和 Icon两个路径即可
```shell
[Desktop Entry]
Name=Obsidian
Exec=AppRun --no-sandbox %U
Terminal=false
Type=Application
Icon=obsidian
StartupWMClass=obsidian
X-AppImage-Version=0.14.2
Comment=Obsidian
MimeType=x-scheme-handler/obsidian;
Categories=Office;
```
！要将.desktop文件也赋予可执行权限，方法同上

>note:尝试好几次失败的原因可能是引用了中文路径，之前放在“下载”文件夹中不成功，放在根目录后成功了

```ad-warning
要将.desktop文件也赋予可执行权限，方法同上
```

```ad-note
尝试好几次失败的原因可能是引用了中文路径，之前放在“下载”文件夹中不成功，放在根目录后成功了
```

