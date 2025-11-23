down:: [[rram model仿真]]

✅3x3 同样的值的 crossbar 计算
🍅自定义输入信号发生器
💡同样的电阻值比较多可以同一时间开启写入
❓存在严重漏电无法完全关断 2025.2.12

![](http://cdn.ljc0606.cn/obsidian/202502111752968.png)
预计结果 500m  500mV/R(620.7k) x 3 = I(2.41uA)?  498.8mV(real)/498.629mV(predict)
![](http://cdn.ljc0606.cn/obsidian/202502122342031.png)
RRAM 电压--阻值对应
![](http://cdn.ljc0606.cn/obsidian/202502122344705.png)

2025.3.25 解决漏电问题：
1. MOS 管漏电是由于源漏存在高场效应导致的，而这是因为电路存在正反馈产生的大电压导致
2. 上述电路问题是由于运算放大器不够理想（更换 functional 中的理想 op）
3. 以及上述电路引入了 MOS 管做开关导致电路不收敛，所以在此基础上添加了负反馈通路，计算变为（A+1）x+b=0

![](http://cdn.ljc0606.cn/obsidian/202503251608325.png)
![](http://cdn.ljc0606.cn/obsidian/202503251608444.png)


