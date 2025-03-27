三种求解思路：数字电路、模拟电路、忆阻器
确定精度？分辨率？
## 模拟电路
### 纯 MOS
[基于MOSFET的平方根计算模拟电路 - 模拟电子技术 - 电子工程网](https://www.eechina.com/thread-3068-1-1.html)
不好实际验证，误差基于 MOS 管
### 基于 op
[OP297GPZ平方根放大器的典型应用, 使用 Analog Devices 的 OP297GPZ 的参考设计 ———EEWorld参考设计中心](https://www.eeworld.com.cn/RDesigns_detail/49437)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411142204982.png?token=ALRC6IXCQK5HHLU46NAKAHDHGYB3K)
误差 0.1%
问题：输入信号频率，供电电压高
好处：方便打板
### 基于 op 和乘法器
[op除法，开平方电路分析-CSDN博客](https://blog.csdn.net/gtkknd/article/details/90017431)

## 数字电路
[三种常见平方根算法的电路设计及Verilog实现与仿真\_verilog开根号-CSDN博客](https://blog.csdn.net/weixin_44699856/article/details/130438117)

二分法和逐次逼近法一致，牛顿法需要除法器
二分思路对于一个 2^n 的输入，需要最多计算 23 次得到结果

## fast root 算法
### First approximation of the result
0.1%误差
基于牛顿法，利用浮点算法原理，和导数快速计算，准确率为

The calculation of ![{\textstyle y={\frac {1}{\sqrt {x}}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/f611da033d0c2da1a553defb74c620b85078d75c) is based on the identity

![{\displaystyle \log _{2}(y)=-{\tfrac {1}{2}}\log _{2}(x)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/18e62b3fe809236cc713e488e0f630c95d279b71)

Using the approximation of the logarithm above, applied to both ![{\displaystyle x}](https://wikimedia.org/api/rest_v1/media/math/render/svg/87f9e315fd7e2ba406057a97300593c4802b53e4) and ![{\displaystyle y}](https://wikimedia.org/api/rest_v1/media/math/render/svg/b8a6208ec717213d4317e666f1ae872e00620a0d), the above equation gives:

IyL−(B−σ)≈−12(IxL−(B−σ))![{\displaystyle {\frac {I_{y}}{L}}-(B-\sigma )\approx -{\frac {1}{2}}\left({\frac {I_{x}}{L}}-(B-\sigma )\right)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/9c3b5cfea3f438feadc80a53a864d8d496525b6d)

Thus, an approximation of Iy![{\displaystyle I_{y}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/6f3eedd0871fd35e1fd8992c53812cd60d1b07c2) is:

Iy≈32L(B−σ)−12Ix![{\displaystyle I_{y}\approx {\tfrac {3}{2}}L(B-\sigma )-{\tfrac {1}{2}}I_{x}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/b09b55dbc5d4fde5e3cdae421e0e3d0c68eca148)

which is written in the code as

i  = 0x5f3759df - ( i >> 1 );

The first term above is the magic number

32L(B−σ)=0x5F3759DF![{\displaystyle {\tfrac {3}{2}}L(B-\sigma )={\mathtt {0x5F3759DF}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/42e9d3393251d29a279ab74985890c38bac8722c)

from which it can be inferred that σ≈0.0450466 ![{\displaystyle \sigma \approx 0.0450466}](https://wikimedia.org/api/rest_v1/media/math/render/svg/5e8f32ed6a6845d85d5dae5e1553bc8dc1f2429d). The second term, 12Ix ![{\displaystyle {\frac {1}{2}}I_{x}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/12da760239ed15e02e094e0a3871bd103564302e), is calculated by shifting the bits of Ix ![{\displaystyle I_{x}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/eed0dfcd8b1ce6eb2e6b7fbc426b895507d53004) one position to the right.
[[27]] (https://en.wikipedia.org/wiki/Fast_inverse_square_root#cite_note-FOOTNOTEHennesseyPatterson1998305-31)

## # 利用 CORDIC 算法计算平方根
[Site Unreachable](https://zhuanlan.zhihu.com/p/336572351)