# 全连接网络的局限性
图片 32*32*3 时，单层全连接神经网络节点为500时
参数个数 = 32 * 32 * 3 * 500 + 500 = 约150w

图片 600 * 600 * 1 的图片  如果三层神经网络，分别为300, 200, 100
参数个数 =  600 * 600 * 300 + 300 * 200 + 200 * 100 = 约1.08亿

参数过多会导致
1. 计算速度减慢
2. 过拟合

解决方案：
我们需要更合理的结构来有效减少参数个数


# 卷积神经网络
## 提出
1962年 Hubel和Wiesel 通过对猫的视觉皮层细胞的研究，提出了感受野(receptive field)的概念。
视觉皮层的神经元就是局部接受信息的，只受某些特定区域刺激的响应。而非对全局图像的感知。
1984年 FuKushima 基于感受野提出了神经认知机(neocognitron)
CNN 是神经认知机的推广

## 结构
CNN 是一个多层的神经网络，每层由多个二维平面组成，其中每个平面由躲个独立神经元组成。

输入层 -> （卷积层+ -> 降采样层?)+ -> 全连接层 -> 分类器层(如softmax层) -> 分类结果
其中卷积层一般最多连续使用三个
1. 输入层：将每个元素代表一个特征节点输入到网络中。
2. 卷积层：*卷积运算的主要目的是使原信号特征增强，并降低噪音
3. 降采样层：*降低网络训练参数及模型的过拟合程度。
又称池化层(pooling)，通俗来说就是把高分辨率图像转为低分辨率图像。
4. 全连接层：对生成的特征进行加权
5. softmax层：获得当前样例属于不同类别的概率

## 什么是卷积
有一个5*5的矩阵(权值矩阵)，对一个3*3的矩阵(矩阵核)进行卷积运算
1. 点积
矩阵的点积运算。即矩阵*矩阵，将对应位置相乘再相加，得到一个数字的操作。
我们取出5*5的第一个3*3矩阵进行计算。
2. 滑动窗口
将 3 * 3 权值矩阵向右滑动一格(步长=1)
3. 重复操作
即重复“点积+滑动”直到输出矩阵被填满

操作后5*5的矩阵被降为3*3了


#### 卷积 vs 全连接
对于5*5的输出3*3 
全连接需要的参数个数为 25 * 9 = 225 个

卷积层需要参数 9 个。

对比得出卷积具有特点：
- 局部连接
每个输出特性不用查看每个输入特征，而只需要查看部分输入特征
- 权值共享
卷积核在图像上滑动过程中保持不变


边缘可能被裁减丢失数据，如何处理？
答：用0填充。通俗说，就是外围加一圈0

## 多通道卷积
用多个卷积核提取不同的特征。

1. 每个通道使用一个卷积核进行卷积操作。(每个卷积核提取一种特征。)
2. 这些特征图相同位置的值相加，生成一张特征图。
3. 加偏置。



## 池化
计算图像一个区域上的某个特定特征的平均值或最大值。

思想：相邻像素点往往想表达相同信息

**卷积**的作用是探测上一层特征的局部连接
使原信号特征增强，并降低噪音

**池化**的作用是在语义上，把相似的特征合并起来，从而达到降维目的
减少数据处理量的同时，保留有用信息。


降维后，不仅具有低得多的维度，同时还能改善结果(不容易过拟合)

## 步长
卷积核在图片上移动的格数

原始图片[N1,N1]，卷积核 [N2,N2] 步长S。
那么卷积之后图像 [(N1-N2)/S + 1 ,(N1-N2)/S + 1]

有一些研究提出，直接提高步长，降维效果比池化好。所以有些模型未使用池化


## TensorFlow卷积函数
定义在 tensorflow/python/opts/nn_impl.py nn_ops.py中

tf.nn.conv2d(input, filter, strides, padding, use_cudnn_on_gpu=None, name=None)
- input 输入数据。是一个4维张量 ([batch, in_height, in_width, in_channels]) 样本个数，高，宽，通道(彩色=3 灰度=1) 要求类型为float32 或float64
- filter 卷积核。是一个4维张量 ([filter_height, filter_width, in_channels, in_channels]),高，宽，输入通道数，输出通道数
- strides 图像每一维的步长。是一个1维张量 长度为4
- padding same 进行0填充 / valid  不填充
- use_cudnn_on_gpu 是否启用cudnn加速 bool类型
- name 该操作的名称
- 返回值 返回一个tensor 即feature map

tf.nn.depthwise_conv2d(input, filter, strides, padding, name=None)
tf.nn.separable_conv2d(input, depthwise_filter, pointwise_filter, strides, padding, name=None)
等等

## TensorFlow池化函数
定义在 tensorflow/python/opts/nn.py gen_nn_ops.py中

最大池化 tf.nn.max_pool(value, ksize, strides, padding, name=None)
平均池化 tf.nn.avg_pool(value, ksize, strides, padding, name=None)
等等

池化层一般位于卷积层之后
- value 输入图像，通常是conv2d输出的feature map 即 [batch, height, width, channels]
- ksize 池化窗口的大小，一般不对batch和channels池化，所以一般是[1,height, width ,1]
- strides 图像每一维的步长
- padding 同卷积函数padding
- name 同卷积函数
- 返回值  返回一个tensor

