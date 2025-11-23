[[lesson+notes#EDA notes]]
### 键位记录
按键	作用
i	添加器件
W	进行连线
L	设置连线标号
Q	修改器件参数
X	保存文件
F	显示完整原理图
C	复制器件
E	进入子模块或下层原理图
Ctrl + E	进入顶模块或上层原理图
Shift + M	移动器件
Shift + N	添加普通文字
鼠标右键	移动器件时旋转

#### 模块层次操作
器件是组件schematic的最小单元

e –> 以只读模式在当前窗口中进入对应组件的下一层

Shift + e –> 以编辑模式在当前窗口中进入对应组件的下一层

Ctrl + e –> 返回当前组件的上一层（以new tab模式时会关闭新打开的tab）

Ctrl + TAB –> 在多个 tab 中切换

#### reference link
[模拟IC设计中的软件操作：Cadence Virtuoso Schematic 电路原理图编辑技巧及其相关快捷键 - 知乎](https://zhuanlan.zhihu.com/p/574080087)

### 激励信号设置
#### vpulse
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230417160056.png)
#### vpwl

#### switch
[cadence Virtuoso ADE原理图AnalogLib库中的switch使用 - 大学生视野 - 博客园](https://www.cnblogs.com/icDesigner/p/14582490.html)
![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920131726.png)
A,B: 等效于一个电阻;
C,D: 等效于控制开关(CD 间的控制电压控制 AB 的断开或闭合);
open switch resistance: 开关断开状态下的等效电阻(AB 之间);
close switch resistance: 开关闭合状态下的等效电阻(AB 之间);
open voltage: 断开开关所需的电压值(CD 间电压低于该值，则 AB 间处于断开状态);
closed voltage: 闭合开关所需的电压值(CD 间电压高于该值，则 AB 间处于闭合状态);

有 W 标记的是正端，正负端不要接错
open voltage 和 closed voltage 设置的值不能相同
（Relay on and off thresholds ('vt2' and 'vt1') must not be the same value
一般设置 open voltage 小于 closed voltage 的值

### snippets
[cadence virtuoso快速查看mos管region【多种方法】_virtuoso region_芯宝典的博客-CSDN博客](https://blog.csdn.net/qq_40007892/article/details/119568781#:~:text=%E6%B3%951%EF%BC%9A%20%E4%BB%BF%E7%9C%9F%E5%AE%8C%E6%88%90%E4%B9%8B%E5%90%8E%EF%BC%8CADE-Results-Print-DC%20operating%20Points%EF%BC%8C%E7%82%B9%E5%87%BB%E6%83%B3%E8%A6%81%E6%9F%A5%E7%9C%8B%E7%9A%84mos%E7%AE%A1%EF%BC%8C%E5%9C%A8%E5%BC%B9%E5%87%BA%E7%9A%84%E8%A1%A8%E4%B8%AD%E6%9F%A5%E6%89%BEregion%EF%BC%8C%20%E7%BC%BA%E7%82%B9%EF%BC%9A%20%E4%B8%80%E6%AC%A1%E5%8F%AA%E8%83%BD%E6%9F%A5%E7%9C%8B%E4%B8%80%E4%B8%AAmos%E7%AE%A1%EF%BC%8C%E5%AE%B9%E6%98%93%E5%BF%98%E8%AE%B0%EF%BC%8C%E7%94%B5%E8%B7%AF%E8%A7%84%E6%A8%A1%E5%A4%A7%E4%BA%86%E4%B9%8B%E5%90%8E%E6%93%8D%E4%BD%9C%E4%B8%8D%E6%96%B9%E4%BE%BF%20%E6%B3%952%EF%BC%9A,DC%E4%BB%BF%E7%9C%9F%E5%AE%8C%E6%88%90%E4%B9%8B%E5%90%8E%EF%BC%8C%E5%9C%A8ADE%E4%B8%AD%E7%82%B9%E5%87%BBresults-Circuit%20Conditions...%2C%E5%BC%B9%E5%87%BAresults%EF%BC%9Acircuit%20conditions%E5%AF%B9%E8%AF%9D%E6%A1%86%EF%BC%8C%E5%8F%AF%E4%BB%A5%E6%A0%87%E6%B3%A8%E5%87%BA%E7%94%B5%E8%B7%AF%E5%9B%BE%E4%B8%AD%E9%A5%B1%E5%92%8C%E5%8C%BABJT%E6%88%96%E7%BA%BF%E6%80%A7%E5%8C%BAmos%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%B7%B1%E8%AE%BE%E7%BD%AE%E6%A0%87%E6%B3%A8%E6%9D%A1%E4%BB%B6%E3%80%82%20%E7%82%B9%E5%87%BBsaturation%20%3CBJT%3E%20or%20Linear%20%3CMOS%3E%E5%90%8E%E7%9A%84%E5%B0%8F%E6%96%B9%E5%9D%97%E9%80%89%E9%A1%B9%E6%A1%86%EF%BC%8C%E9%80%89%E6%8B%A9%E6%A0%87%E6%B3%A8%E9%A2%9C%E8%89%B2%EF%BC%8C%E7%84%B6%E5%90%8E%E7%82%B9%E5%87%BB%E5%8F%B3%E4%BE%A7results-annotate%E4%B8%AD%E7%9A%84place%EF%BC%8C%E5%B0%B1%E5%8F%AF%E4%BB%A5%E5%9C%A8%E7%94%B5%E8%B7%AF%E5%9B%BE%E4%B8%AD%E7%9C%8B%E5%88%B0%E6%89%80%E6%9C%89%E4%BD%8D%E4%BA%8E%E9%A5%B1%E5%92%8C%E5%8C%BABJT%E6%88%96%E7%BA%BF%E6%80%A7%E5%8C%BAmos%E3%80%82)
#### result print
region 为 0,1,2,3,4 依次代表 [MOS管](https://so.csdn.net/so/search?q=MOS%E7%AE%A1&spm=1001.2101.3001.7020)工作在截止区、线性区、饱和区、亚阈值区、breakdown


### 画线
schematic 中画斜线：按 w，鼠标点一下选中线的起点，按 F3，draw mode 选择斜线，angle 可以选择 lock 到 45°或者 any，线的形状和颜色都可以选择，点状 dots、粗细等等都在 line style 中可以选。

### 波形图

4、在曲线上标两点求斜率

![](https://pic4.zhimg.com/80/v2-c7e027a348cda44121326c726bb3b8ff_720w.webp)

效果如下：

![](https://pic2.zhimg.com/80/v2-493286421fb32edf42d9c6c143384b09_720w.webp)

5、选定两个标记(Marker)求增量

如果放下了一堆标记（点、垂直、水平），则可以选择其中的 2 个或更多标记（使用 Ctrl 单击选择多个标记），然后按 Shift-D 获取所有选定标记之间的增量标记。这样做很酷的事情是，您可以将点标记与垂直或水平标记混合，以获取从点到线的增量值。

6、创建增量标记

这可能会是你创建增量标记的最常用方法。只需选择一个点，垂直或水平标记（M，V，H），然后无论您在何处按 D 键，您都会获得该类型的第二个标记和之间的增量。由于当您创建点，垂直或水平标记时，它保持选中状态，因此您可以使用序列 M，D，D，D…或 V， D， D， D…或 H， D， D， D…只需按几下键即可获取多个标记，这些标记之间具有增量值。

比如我在得到一个带隙电压时，想看最低值和最高值之间的电压差，就可以先点一下 H，拖到最高处，再点一下 D，此时两个横线是重合的，需要你手动把它拖出来，拉到你想要的地方。然后就会有Δy 显示(有可能显示到图像外面，需要把它拉回屏幕)，如下：

### calculator
[【工具小技巧】Cadence Virtuoso Calculator Function Panel计算器函数功能介绍（持续更新……）\_virtuoso计算器\_喝喝喝水的博客-CSDN博客](https://blog.csdn.net/m0_57592021/article/details/128755499#:~:text=signal%E4%B8%80%E6%A0%8F%E8%BE%93%E5%85%A5%E6%B3%A2%E5%BD%A2%E5%87%BD%E6%95%B0%EF%BC%8Cinitial%20value%20type%E9%80%89%E6%8B%A9y%EF%BC%8C%E8%A1%A8%E6%98%8E%E8%AE%BE%E7%BD%AE%E7%9A%84%E6%98%AFy%E8%BD%B4%E7%9A%84%E6%95%B0%EF%BC%8C%E5%AF%B9%E4%BA%8E%E4%B8%8A%E5%8D%87%E6%B2%BF%EF%BC%88%E8%AE%A1%E7%AE%97%E4%B8%8A%E5%8D%87%E6%97%B6%E9%97%B4%EF%BC%89%EF%BC%8Cinitial%2Ffinal,value%E5%80%BC%E5%88%86%E5%88%AB%E4%B8%BA%E6%9C%80%E5%B0%8F%E5%80%BC%E5%92%8C%E6%9C%80%E5%A4%A7%E5%80%BC%EF%BC%88%E8%BF%99%E9%87%8C%E8%AE%BE%E7%BD%AE%E4%B8%BA0%E5%88%B0avdd%EF%BC%89%EF%BC%8C%E5%AF%B9%E4%BA%8E%E4%B8%8B%E9%99%8D%E6%B2%BF%EF%BC%88%E8%AE%A1%E7%AE%97%E4%B8%8B%E9%99%8D%E6%97%B6%E9%97%B4%EF%BC%89%EF%BC%8Cinitial%2Ffinal%20value%E5%80%BC%E5%88%86%E5%88%AB%E4%B8%BA%E6%9C%80%E5%A4%A7%E5%80%BC%E5%92%8C%E6%9C%80%E5%B0%8F%E5%80%BC%EF%BC%88%E8%BF%99%E9%87%8C%E8%AE%BE%E7%BD%AE%E4%B8%BAavdd%E5%88%B00%EF%BC%89%EF%BC%8Cpercent%20high%2Flow%E4%B8%BA%E5%8F%96%E7%9A%84%E4%B8%8A%E5%8D%87%2F%E4%B8%8B%E9%99%8D%E6%B2%BF%E7%9A%84%E8%8C%83%E5%9B%B4%EF%BC%8C%E8%BF%99%E9%87%8C%E5%8F%9610%25-90%25%EF%BC%8C%E5%8D%B3avdd%E7%9A%8410%25%E5%88%B090%25%E3%80%82)

region 查看方法：
[cadence virtuoso快速查看mos管region【多种方法】\_virtuoso region\_芯宝典的博客-CSDN博客](https://blog.csdn.net/qq_40007892/article/details/119568781)