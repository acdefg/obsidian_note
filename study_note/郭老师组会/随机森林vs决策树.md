## anaconda
### 安装
[Index of /anaconda/archive/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)
官网点不动 qs
11.05 安装参考：
[Anaconda下载和安装指南（超全） - 知乎](https://zhuanlan.zhihu.com/p/359471207)
install for all user --x
add anaconda path --√
[如何彻底卸载Anaconda？_Lord_Bao的博客-CSDN博客_卸载anaconda](https://blog.csdn.net/Lord_Bao/article/details/114170382)
Xs边装边卸
### 环境配置

```shell
activate python38
```

```shell
pip install numpy matplotlib pandas scipy
```

```shell
pip install scikit-learn
```

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221105150521.png)

### Jupyter

常用快捷键：
Shift+Enter ，执⾏本单元代码，并跳转到下⼀单元
Ctrl+Enter ，执⾏本单元代码，留在本单元
Esc+m，切换为 Markdown 模式，单元中内容保存为文本

安装插件：
```shell
activate 虚拟环境名字

pip install jupyter_contrib_nbextensions -i

jupyter contrib nbextension install --user --skip-running-check

```
然后重启 jupyter notebook

## 原理们
[一文看懂随机森林 - Random Forest（4个实现步骤+10个优缺点）](https://easyai.tech/ai-definition/random-forest/)
[一文看懂决策树 - Decision tree（3个步骤+3种典型算法+10个优缺点）](https://easyai.tech/ai-definition/decision-tree/)

### 随机森林

#### 树的生成
有了树我们就可以分类了，但是森林中的每棵树是怎么生成的呢？

每棵树的按照如下规则生成：

1）如果训练集大小为N，对于每棵树而言，随机且有放回地从训练集中的抽取N个训练样本（这种采样方式称为bootstrap sample方法），作为该树的训练集；

从这里我们可以知道：每棵树的训练集都是不同的，而且里面包含重复的训练样本（理解这点很重要）。

**为什么要随机抽样训练集？（add @2016.05.28）**

如果不进行随机抽样，每棵树的训练集都一样，那么最终训练出的树分类结果也是完全一样的，这样的话完全没有bagging的必要；

**为什么要有放回地抽样？**（add @2016.05.28）****

我理解的是这样的：如果不是有放回的抽样，那么每棵树的训练样本都是不同的，都是没有交集的，这样每棵树都是"有偏的"，都是绝对"片面的"（当然这样说可能不对），也就是说每棵树训练出来都是有很大的差异的；而随机森林最后分类取决于多棵树（弱分类器）的投票表决，这种表决应该是"求同"，因此使用完全不同的训练集来训练每棵树这样对最终分类结果是没有帮助的，这样无异于是"盲人摸象"。

2）如果每个样本的特征维度为M，指定一个常数m<<M，随机地从M个特征中选取m个特征子集，每次树进行分裂时，从这m个特征中选择最优的；

3）每棵树都尽最大程度的生长，并且没有剪枝过程。
一开始我们提到的随机森林中的“随机”就是指的这里的两个随机性。两个随机性的引入对随机森林的分类性能至关重要。由于它们的引入，使得随机森林不容易陷入过拟合，并且具有很好得抗噪能力（比如：对缺省值不敏感）。


### 鸢尾花数据集
iris 以鸢尾花的特征作为数据来源，常用在分类操作中。该数据集由 3 种不同类型的鸢尾花的各 50 个样本数据构成。其中的一个种类与另外两个种类是线性可分离的，后两个种类是非线性可分离的。

该数据集包含了4个属性：

& Sepal.Length（花萼长度），单位是cm;

& Sepal.Width（花萼宽度），单位是cm;

& Petal.Length（花瓣长度），单位是cm;

& Petal.Width（花瓣宽度），单位是cm;

种类：Iris Setosa（山鸢尾）、Iris Versicolour（杂色鸢尾），以及 Iris Virginica（维吉尼亚鸢尾）。

![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221106125732.png)

```verilog
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
import pandas as pd
import numpy as np

iris = load_iris()
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['is_train'] = np.random.uniform(0, 1, len(df)) <= .75
df['species'] = pd.Categorical.from_codes(iris.target, iris.target_names)
df.head()

train, test = df[df['is_train']==True], df[df['is_train']==False]

features = df.columns[:4]
clf = RandomForestClassifier(n_jobs=2)
y, _ = pd.factorize(train['species'])
clf.fit(train[features], y)

preds = iris.target_names[clf.predict(test[features])]
pd.crosstab(test['species'], preds, rownames=['actual'], colnames=['preds'])
```


### 存在问题
每种特性背后的具体原理不清楚
一种统计学的知识不太牢固
