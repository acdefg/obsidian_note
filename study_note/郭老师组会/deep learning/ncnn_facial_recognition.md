In the context of "Nihao's Convolutional Neural Network", "Nihao" is not a name but rather an acronym for "N eural network I nference H ardware A cceleration O ptimization". The creators of NCNN chose this name to represent their focus on optimizing neural network inference performance on a variety of hardware platforms, particularly mobile and embedded devices.

[Site Unreachable](https://blog.csdn.net/weixin_46236212/article/details/122570929)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230311090827.png)


## questions
### TF-SLIM
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230312095027.png)
TF-Slim 是一个轻量级的库，用于在 TensorFlow 中定义，训练和评估复杂的模型。tf-slim 的组件可以与本机 tensorflow 以及其他框架自由混合。
Tensorflow2.x 版本较 1.x 版本有了很大的变动，以使 TensorFlow 用户更加高效。其中 tf.contrib 被完全弃用了是 2.x 版本的一个重大的变化，但 import tensorflow.contrib.slim as slim slim 作为一个高级封装，已经在很多之前的版本中广泛使用。现在大部分的源码还是以 tensorflow1.x 版本为基础写的，这导致了一些已经在 2.x 版本中移除的模块无法使用。
经查询现有的解决方案，大部分采用了降低版本的方法，如果想采用此方法可以自己去查询。
因为不想采用降低版本的方法进行解决，经过搜索在 github 中查询到此信息
链接: tf.contrib.slim is not worked in tensorflow 2.0 what is the alternative for that?.
### 怎么在 github 上下载单独文件
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230312100138.png)
