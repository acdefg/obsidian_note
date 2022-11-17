[gcc编译优化-O0 -O1 -O2 -O3 -OS解析_奔跑的码农的博客-CSDN博客_gcc o2](https://blog.csdn.net/wuxing26jiayou/article/details/96132721) 选项内容具体描述
[汇编视角:不同优化级别下的GCC行为分析 | 海森的博客](https://hisenz.com/post/%E6%B1%87%E7%BC%96%E8%A7%86%E8%A7%92-%E4%B8%8D%E5%90%8C%E4%BC%98%E5%8C%96%E7%BA%A7%E5%88%AB%E4%B8%8B%E7%9A%84GCC%E8%A1%8C%E4%B8%BA%E5%88%86%E6%9E%90/) 👍


选择 memcpy 作为例子是因为它的实现代码简单, 但是涉及了传参, 条件判断和循环, 是逻辑密集型的代码, 能很好的体现 gcc 在逻辑上的优化。
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211171818644.png)
