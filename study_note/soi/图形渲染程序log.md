[GitHub - mmmovania/opencloth: A collection of source codes implementing cloth simulation algorithms in OpenGL](https://github.com/mmmovania/opencloth)
有一些 demo，但是需要 visual studio
或者可以 windows 的打开看看那
status: waiting

[GitHub - dilevin/CSC417-a4-cloth-simulation: Cloth simulation using co-rotational linear elasticity](https://github.com/dilevin/CSC417-a4-cloth-simulation)
status: quit
log: 
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308241317665.png)
按道理来说能检测到路径，但是就是打不开，有毒了
可以试试是不是 ig.. 库安装的问题

[GitHub - MauriceGit/Cloth\_Simulation: Cloth-Visualization via particle-simulation.](https://github.com/MauriceGit/Cloth_Simulation)

### eol-cloth
[GitHub - sueda/eol-cloth: Eulerian-on-Lagrangian Cloth Simulation](https://github.com/sueda/eol-cloth)
那个 cmakelist 个根本不适用 ubuntu，改了一点不想改了，重写一个

### DART
[GitHub - dartsim/dart: DART: Dynamic Animation and Robotics Toolkit](https://github.com/dartsim/dart)
[Biped — DART: Dynamic Animation and Robotics Toolkit 7.0.0-alpha20230101 documentation](https://dart.readthedocs.io/en/latest/user_guide/tutorials/biped.html)
好像是一个完整的库
status：昨天打开 cmake all 在之后就没停下来
#### log
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309010855320.png)
应该是有错误在不断重试

[GitHub - hbertiche/NeuralClothSim](https://github.com/hbertiche/NeuralClothSim)
没看懂具体干啥的，但是可以试试
深度学习做的

[Human Motion Diffusion Model](https://guytevet.github.io/mdm-page/)
深度学习做的

### sheen
[GitHub - sciecode/sheen: GPGPU cloth simulation](https://github.com/sciecode/sheen)
java 写的
效果不错
📍：😒 quit

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311743254.png)

[[vscode_ubuntu_javajs]]
合着我折腾一天配置 java 环境，跑出来就和 github 点进去一样
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011152845.png)
啥也没有，无语死了
不应该阿，代码和好像写了相关的东西 emmmmmm
[• Atomize •](https://sciecode.com/)
另一个网站可以，一下子就运行成功了，哭哭
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011439278.png)

只加载出来了背景，加载方式不太对劲，另外一个用其他在线平台验证的，本地也跑不出来，canvas 的问题？先放弃了。。。。。


[GitHub - JUSTIVE/GPU-Cloth-Simulation: GPU Mass-Spring Simulation Cloth in Unity](https://github.com/JUSTIVE/GPU-Cloth-Simulation)
看起来不错
test on windows
给了一堆测试向量的参考值
📍status：waiting
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311738274.png)

#### ClothSimulation ✔️
[GitHub - johnBuffer/ClothSimulation: Basic cloth simulation using Verlet integration](https://github.com/johnBuffer/ClothSimulation)
代码完整，调用库少
简陋
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311646565.png)
👍 status: success
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311716756.png)

Left click 	Move view
Wheel 	Zoom
Right click 	Move cloth
Middle click 	Cut cloth

这个差不多，java 新写的
[GitHub - dissimulate/Tearable-Cloth: A tearable cloth simulation using vertlet integration.](https://github.com/dissimulate/Tearable-Cloth)
### environment
#### SFML
[SFML and Linux (SFML / Learn / 2.5 Tutorials)](https://www.sfml-dev.org/tutorials/2.5/start-linux.php)

**▶️ test pass**

📢 **pay attention**
You must then link the compiled file to the SFML libraries in order to get the final executable. SFML is made of 5 modules (system, window, graphics, network and audio), and there's one library for each of them.
To link an SFML library, you must add "-lsfml-xxx" to your command line, for example "-lsfml-graphics" for the graphics module (the "lib" prefix and the ".so" extension of the library file name must be omitted).
```
g++ main.o -o sfml-app -lsfml-graphics -lsfml-window -lsfml-system
```

#### eigen

[ubuntu安装Eigen\_ubuntu22安装eigen\_ClaireQi的博客-CSDN博客](https://blog.csdn.net/wangxiao7474/article/details/103422616)

```
sudo apt-get install libeigen3-dev
```
```
# /usr/include
sudo cp -r /usr/include/eigen3/Eigen /usr/include
# /usr/local/include
sudo cp -r /usr/local/include/eigen3/Eigen /usr/local/include
```
测试：
[Eigen的介绍、安装与入门操作 - 知乎](https://zhuanlan.zhihu.com/p/462494086)

#### opengl
[Ubuntu下搭建OpenGL开发环境（GLFW\_3.3.1 + GLM\_0.9.9 + GLAD）\_RoboticsLearner的博客-CSDN博客](https://blog.csdn.net/l1216766050/article/details/102787618)

[Linux（Ubuntu）使用 sudo apt-get install 命令安装软件的目录在哪？（已解决） - Memory4Young - 博客园](https://www.cnblogs.com/memory4young/p/where-is-sudo-apt-get-install-package-file-path.html)

测试一直报错
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011712246.png)
改了半天搜索路径一直不对，不想在 cmake 在中额外加搜索路径，直接找到这个文件复制进来
```
sudo find / -name libglfw3.a
cp /usr/local/lib/libglfw3.a .  //路径根据具体情况改
```

#### gdb（待整理）
[GDB调试入门指南 - 知乎](https://zhuanlan.zhihu.com/p/74897601)
[【Linux】GDB调试教程（新手小白）\_爪可摘星辰的博客-CSDN博客](https://blog.csdn.net/lovely_dzh/article/details/109160337#t12)

