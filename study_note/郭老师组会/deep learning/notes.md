[Site Unreachable](https://blog.csdn.net/icebearand77/article/details/123094530) colab 数据集上传
[构建自定义人脸识别数据集的三种训练方法 -ATYUN](http://www.atyun.com/22853.html)

解决方法一：根据训练过程的 epoch 次数设定 dataset_train = dataset_train .repeat(epoch)

解决方法二：仍然需要进行repeat（）设定，但epoch参数为空，即：

                       dataset_train = dataset_train .repeat()

                       当repeat()参数为空时，意思是重复无数遍，永远不会有读取不到数据batch的情况。
[Tensorflow dataset.repeat（）的使用_seuzhouchenglong的博客-CSDN博客](https://blog.csdn.net/seuzhouchenglong/article/details/104047784)



