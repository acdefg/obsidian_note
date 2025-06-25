[最好的OpenGL教程之一\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1MJ411u7Bc/?spm_id_from=333.337.search-card.all.click&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)

# P1：Why learning Opengl
简单、跨平台，相比于其他 API：Vulkan、Direct3D.... 更适合新手，是一种控制 GPU 的规范，可以访问 GPU 的图形 API，opengl 是由显卡制造产商实现的，在 GPU 驱动中


# P2：设置 Opengl 和创建窗口

实现一个跨平台可以运行的窗口创建程序，`GLFW` 库可以帮助实现这一点

## GLFW 的安装
这里为了方便直接下载了 windows 的预编译版本
GLFW 官方网址： [An OpenGL library \| GLFW](https://www.glfw.org/)
右上角导航栏选择：Download，打开后 Windows 用户可选择 32-bit Windows binaries，linux 只能下载源码或者通过 `sudo apt install libglfw3-dev libglfw3` 安装 glfw
>选择 32-bit 在创建项目时，只能创建 32bit 的项目，至于选择 32bit的原因，作者没有明说，提到在 64bit 环境中也能运行 32bit 代码，猜测可能兼容性更好

打开后：找到 include 文件夹，和最新的 lib-vc 版本
![](http://cdn.ljc0606.cn/obsidian/202506252234202.png)

在项目目录下建立 Dependencies 文件夹，存放上述两个文件夹，删掉 lib 中的 `.dll` 和后缀 `dll` 的 lib 文件，这个用于 动态编译*
配置：
```
项目属性 -> C/C++ -> 附加包含目录 -> 添加 $(SolutionDir)Dependencies\GLFW\include
项目属性 -> 链接器 -> 常规 -> 附加库目录 -> 添加: $(SolutionDir)Dependencies\GLFW\lib-vc2022
项目属性 -> 链接器 -> 输入 -> 附加依赖项 -> 添加: glfw3.lib
```
