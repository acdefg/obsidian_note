### visual studio 安装
[Visual Studio 2022安装教程(非常详细)，从零基础入门到精通，看完这一篇就够了（附安装包）-CSDN博客](https://blog.csdn.net/qq_44005305/article/details/132295064#:~:text=1.VS%E4%B8%8B%E8%BD%BD%E5%AE%98%E7%BD%91%EF%BC%9A%20%E7%82%B9%E8%BF%99%E9%87%8C%201%202.%E7%82%B9%E8%BF%9B%E5%8E%BB%E4%B9%8B%E5%90%8E%E4%BC%9A%E8%87%AA%E5%8A%A8%E4%B8%8B%E8%BD%BDvs.exe%E6%96%87%E4%BB%B6%EF%BC%88%E5%A6%82%E4%B8%8B%E5%9B%BE%E6%89%80%E7%A4%BA%EF%BC%89%EF%BC%9A%202%203.%E4%B8%8B%E8%BD%BD%E5%AE%8C%E6%88%90%E5%90%8E%E5%A6%82%E4%B8%8B%E5%9B%BE%E6%89%80%E7%A4%BA%EF%BC%8C%E7%9B%B4%E6%8E%A5%E7%82%B9%E5%87%BB%E7%BB%A7%E7%BB%AD%E5%8D%B3%E5%8F%AF%E3%80%82%203%204.%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90,IDE%E6%94%BE%E5%9C%A8%E9%BB%98%E8%AE%A4%E4%BD%8D%E7%BD%AE%EF%BC%8C%E8%BF%90%E8%A1%8C%E9%80%9F%E5%BA%A6%E5%8F%AF%E8%83%BD%E4%BC%9A%E6%9B%B4%E5%BF%AB%E4%B8%80%E4%BA%9B%EF%BC%89%205%206.%E5%AE%8C%E6%88%90%E4%B8%8A%E8%BF%B0%E6%93%8D%E4%BD%9C%E7%82%B9%E5%87%BB%E5%AE%89%E8%A3%85%EF%BC%8C%E7%AD%89%E5%BE%85%E4%B8%8B%E8%BD%BD%E5%AE%8C%E6%88%90%E5%8D%B3%E5%8F%AF%E3%80%82%20%E4%BA%8C%E3%80%81%E6%B5%8B%E8%AF%95%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F%201.%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%20%E7%AC%AC%E4%B8%80%E6%AD%A5%EF%BC%9A%E6%89%93%E5%BC%80VS%EF%BC%8C%E7%82%B9%E5%87%BB%E5%88%9B%E5%BB%BA%E6%96%B0%E9%A1%B9%E7%9B%AE%EF%BC%88N%EF%BC%89%20%E7%AC%AC%E4%BA%8C%E6%AD%A5%EF%BC%9A%E7%82%B9%E5%87%BB%E7%A9%BA%E9%A1%B9%E7%9B%AE%20)
官网：
[Visual Studio: 面向软件开发人员和 Teams 的 IDE 和代码编辑器](https://visualstudio.microsoft.com/zh-hans/)
下载，选择开发套件，更改地址，安装
### opengl 配置
有两种：
一种是使用.sln 工程文件配置安装的（感觉需要每个工程都来一遍，不想搞这个）
[VS 2022配置openGL环境（GLFW+GLEW）\_vs配置opengl-CSDN博客](https://blog.csdn.net/FallenChild/article/details/128044052)
一种是提取 include，lib 分别放到对应目录去的
参考这篇：
[VS2022的openGL环境搭建（完整篇） - 知乎](https://zhuanlan.zhihu.com/p/486459964)
glew 要另外安装，差不多一样的步骤

### 注意事项
#### nupengl
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20231129143600.png)
每次新建项目都要安装一下 nupengl.core 和 nupengl.core.redist
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20231129143624.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20231129143635.png)
安装一个另外一个会自动安装

#### glad

在 VS 项目中如果需要用 GLAD，就要把 glad.c 复制到源文件目录下
glad.c 在 `G:\\software\\Opengl\\glad\\src`

#### glew
glGenVertexArrays(1, &VAO);
glGenVertexArrays，VAO 在使用前需要现在 glewInit()之前加上：
```
glewExperimental = GL_TRUE;
```
[关于c ++：openGL：glGenVertexArrays执行访问冲突的位置0x00000000 | 码农家园](https://www.codenong.com/30061443/)
[OpenGL GAO访问冲突（glBindVertexArray）-CSDN博客](https://blog.csdn.net/fan2273/article/details/75072823)

> [!info]+ Note
> 
测试了两组三组代码在 workplace 里面