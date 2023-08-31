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

[GitHub - dartsim/dart: DART: Dynamic Animation and Robotics Toolkit](https://github.com/dartsim/dart)
[Biped — DART: Dynamic Animation and Robotics Toolkit 7.0.0-alpha20230101 documentation](https://dart.readthedocs.io/en/latest/user_guide/tutorials/biped.html)
好像是一个完整的库
status：昨天打开 cmake all 在之后就没停下来

[GitHub - hbertiche/NeuralClothSim](https://github.com/hbertiche/NeuralClothSim)
没看懂具体干啥的，但是可以试试
深度学习做的

[Human Motion Diffusion Model](https://guytevet.github.io/mdm-page/)
深度学习做的


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

