In the context of "Nihao's Convolutional Neural Network", "Nihao" is not a name but rather an acronym for "N eural network I nference H ardware A cceleration O ptimization". The creators of NCNN chose this name to represent their focus on optimizing neural network inference performance on a variety of hardware platforms, particularly mobile and embedded devices.

## facenet 和 mobilenet
FaceNet 和 MobileNet 都是深度学习中比较流行的轻量级卷积神经网络模型，它们有一定的关系，但是在应用场景和目标上有所不同。

FaceNet是由Google研发的一种人脸识别模型，采用了一种称为三重损失函数的训练方法，可以将人脸图像映射为一个高维度的向量表示，从而实现了对人脸的有效识别。FaceNet基于卷积神经网络（CNN）实现，包括多个卷积层和全连接层。

MobileNet则是一种专门针对移动设备和嵌入式系统优化的卷积神经网络模型，旨在实现高效的模型训练和推理。MobileNet通过在卷积操作中引入深度可分离卷积，降低了模型的参数数量和计算复杂度，从而实现了在嵌入式设备上高效的运行。

虽然 FaceNet 和 MobileNet 都是卷积神经网络模型，但是它们的应用场景和目标不同。FaceNet 主要用于人脸识别领域，需要处理复杂的人脸图像，并且需要高精度的识别结果。MobileNet 则主要用于移动设备和嵌入式系统领域，需要在有限的计算资源和存储空间下实现高效的模型推理，通常应用于图像分类、目标检测等任务。

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

我们可以通过安装 tf_slim 模块来替换 contrib 中 slim，安装 tf_slim 模块

```bash
pip install tf_slim
```

```python
import tf_slim as slim
```
### module 'tensorflow.compat.v2.__internal__' has no attribute 'register_load_context_function'
降低 tensorflow 和 keras 的版本并保持一致
更改代码：
`from keras.models import load_model`
`from tensorflow.python.keras.models import load_model`
### OSError: SavedModel file does not exist at:
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230312121303.png)
自定义层，需要在 load_model()里，加上 cuustom_objects，暂时没成功
### 开了代理无法下载包
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230312161919.png)
