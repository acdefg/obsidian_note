[最好的OpenGL教程之一\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1MJ411u7Bc/?spm_id_from=333.337.search-card.all.click&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)

# P1：Why learning OpenGL
简单、跨平台，相比于其他 API：Vulkan、Direct3D.... 更适合新手，是一种控制 GPU 的规范，可以访问 GPU 的图形 API，opengl 是由显卡制造产商实现的，在 GPU 驱动中


# P2：设置 OpenGL 和创建窗口

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

opengl 添加：
项目属性 -> 链接器 -> 输入 -> 附加依赖项 -> 添加: opengl32.lib
原视频中为了保持简洁，一个一个添加 lib，直接搜缺少的函数，然后在 MSDN 上能找到属于哪个lib
```
//直接默认lib
$(CoreLibraryDependencies);%(AdditionalDependencies);glfw3.lib;opengl32.lib
```

## 使用传统 opengl 绘制三角形

```
/* Render here */
glClear(GL_COLOR_BUFFER_BIT);

//Draw triangle
glBegin(GL_TRIANGLES);
glVertex2f(-0.5f, -0.5f);
glVertex2f(0.5f, -0.5f);
glVertex2f(0.0f, 0.5f);
glEnd();
```

# P3：使用现代 OpenGL
由于现代 OpenGL 是硬件驱动实现的非开源代码，需要用 GLEW 库来提供 OpenGL 的 api 接口，是 OpenGL 的扩展，其他库比如 GLUE 是一种 OpenGL 的特殊扩展。这里使用 GLEW。

## GLEW 配置
下载链接：[Just a moment...](https://sourceforge.net/projects/glew/files/glew/2.1.0/glew-2.1.0.zip/download)
在解压出来的文件夹，复制到 `Dependencies` 下面，重命名为 `GLEW`（方便索引），在文件夹中可以找到 `doc/`，有关于如何使用的 html，请阅读这份文档

