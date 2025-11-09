# Research paper

# Abstract

随着 VR 技术和游戏行业的兴起，计算机图形学算法的加速策略也随之不断发展。由于 CPU 和 GPU 的局限性，其中物理引擎计算部分的加速发展受限，相较而言，CPU 不擅长处理图像，GPU 不擅长求解方程。而在软件层面的优化局限于新的迭代算法和预处理。基于 RRAM 阵列加速物理引擎算法…..

# **Algorithm Structure**

![Untitled](Research%20paper%2034781decbe604a0fb61ac2ca78157b1d/Untitled.jpeg)

[关于数量](https://www.notion.so/3e770534b7364541a81cf4ea4e2ea10c?pvs=21)

[compute force](https://www.notion.so/compute-force-3a26102f16f44a5ab122e01e0ca5a0d5?pvs=21)

[collision](https://www.notion.so/collision-9de01a59a217449db92ba0dbbe39d42a?pvs=21)

[ApplyProvotDynamicInverse](https://www.notion.so/ApplyProvotDynamicInverse-573f7207716246878d6e975c85c6e9eb?pvs=21)

# Data structure

spring.size:

spring 信息：即 p1（int32），p2（int32），ks（float16），kd（float16），k（float16），reset_l（float32）其中 x1，x2 为不定值，其余为定值

![Untitled](Research%20paper%2034781decbe604a0fb61ac2ca78157b1d/Untitled%201.jpeg)

points.size:

X（float32），V（float32），F’(x)（float32），F(x)（float32）

# FLOW

for total points :
F = 0
F = F + gravity

（gravity = [0, -9.8 ,0]）

for springs.size :

- **位置差向量 ΔP:**

$$
ΔP=p1−p2=(Xx−Xx,Xy−Xy,Xz−Xz)
$$

- **单位位置差向量 ΔP^**

  $$
  \hat{\Delta \mathbf{P}} = \frac{\Delta \mathbf{P}}{|\Delta \mathbf{P}|} = \frac{(X1_x - X2_x, X1_y - X2_y, X1_z - X2_z)}{\sqrt{(X1_x - X2_x)^2 + (X1_y - X2_y)^2 + (X1_z - X2_z)^2}}
  $$

- **速度差向量 ΔV**

  $$
  \Delta \mathbf{V} = v_1 - v_2 = (V1_x - V2_x, V1_y - V2_y, V1_z - V2_z)
  $$

- **距离 dist**：

  $$
  \text{dist} = |\Delta \mathbf{P}| = \sqrt{(X1_x - X2_x)^2 + (X1_y - X2_y)^2 + (X1_z - X2_z)^2}
  $$

$$
F_{\text{elastic}} = -K_s \cdot (\text{dist} - \text{rest\_length}) \cdot \hat{\Delta \mathbf{P}}
$$

$$
F_{\text{damping}} = K_d \cdot \frac{\Delta \mathbf{V} \cdot \Delta \mathbf{P}}{\text{dist}} \cdot \hat{\Delta \mathbf{P}}
$$

$$
F_{\text{spring}} = F_{\text{elastic}} + F_{\text{damping}}
$$

![Untitled](Research%20paper%2034781decbe604a0fb61ac2ca78157b1d/Untitled%202.jpeg)

![Untitled](Research%20paper%2034781decbe604a0fb61ac2ca78157b1d/Untitled%203.jpeg)

# Ideas

## ARRARY

利用 3*3 阵列计算三维坐标，避免大规模线性方程组求解（精度，灵活性）

原来的电路公式为：

$$
Vout*R+I=0
$$

实际电路中电路稳定时应该为:

$$
(Vout-Vb)*R+I=0
$$

### 其他

定义四种算子

使用多值计算，将 RRAM 分成 8 值，每一个代表八进制的一位

将大的参数拆成同样的几个矩阵的和，分别对应八进制的每一位，最后将结果相加

## CAM

CAM1 中需要存储 spring 信息，即 x1（float32），x2（float32），ks（float16），kd（float16），k（float16），其中 x1，x2 为不定值，其余为定值，只考虑 x1，x2 的存储更新

CAM2 中需要存储 point 信息，即 x（float32），v（float32），f’(x)，f(x)

## Pipeline