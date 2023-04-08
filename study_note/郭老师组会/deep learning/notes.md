[Site Unreachable](https://blog.csdn.net/icebearand77/article/details/123094530) colab 数据集上传
[构建自定义人脸识别数据集的三种训练方法 -ATYUN](http://www.atyun.com/22853.html)

解决方法一：根据训练过程的 epoch 次数设定 dataset_train = dataset_train .repeat(epoch)

解决方法二：仍然需要进行repeat（）设定，但epoch参数为空，即：

                       dataset_train = dataset_train .repeat()

                       当repeat()参数为空时，意思是重复无数遍，永远不会有读取不到数据batch的情况。
[Tensorflow dataset.repeat（）的使用_seuzhouchenglong的博客-CSDN博客](https://blog.csdn.net/seuzhouchenglong/article/details/104047784)

[Site Unreachable](https://blog.csdn.net/m0_57190374/article/details/127373965)  Tensorflow 常见的抑制过拟合方法：数据增强、Dropout、BatchNormalization、正则化
在训练过程中随机旋转图像的预处理层
tf.keras.layers.RandomRotation(
    factor,#一个表示为 2 的小数的浮点数，或一个大小为 2 的元组，表示顺时针和逆时针旋转的上下界。正值表示逆时针旋转，负值表示顺时针旋转。
    fill_mode='reflect',#输入边界以外的点的填充模式{"constant", "reflect", "wrap", "nearest"}
    interpolation='bilinear',#插值模式。支持值:“nearest”，“bilinear”。
    seed=None,
    fill_value=0.0,
    **kwargs
)
在训练过程中随机平移图像的预处理层

 tf.keras.layers.RandomTranslation(
    height_factor,#表示为值的一部分的浮点数，或大小为 2 的元组，表示垂直移动的下限和上限
    width_factor,# 水平移动上下限
    fill_mode='reflect',#输入边界以外的点的填充模式{"constant", "reflect", "wrap", "nearest"}
    interpolation='bilinear',#插值模式。支持值:“nearest”，“bilinear”。
    seed=None,
    fill_value=0.0,
    **kwargs
)

## 测试图片
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230407213637.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230407214526.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230407214540.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230407214548.png)

validation_split=.30, 
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230408205827.png)

![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/f9553b49d58cde32e791b5849b84be0.png)


## debug store
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230407231422.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230407231601.png)
