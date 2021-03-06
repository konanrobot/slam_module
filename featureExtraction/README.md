# FeatureExtraction——Designed by 叶培楚

### 代码内容介绍
通过FeatureExtraction类将特征提取的内容封装起来。

特征类型包括：
1. ORB
2. SIFT
3. SURF
4. AKAZE
5. BRIEF

**PS: 想要使用SIFT, SURF和BRIEF需要安装Opencv-contrib包。**

> 特征提取函数有两种类型：
1. 利用Opencv函数直接对原图像进行特征提取；
2. 对原图像进行简单的分割，之后对每个子图进行特征提取，再将所有特征点汇总。
PS: 第二种方法相对于ORB-SLAM在提取的结果上，几乎没有可比性。毕竟ORB-SLAM是通过对图像进行非常细致的分割，再利用FAST来检测角点，从头到尾都是自己手撸的。我这边是用了OpenCV开源库的函数。若有需要，可以基于此进行拓展。


### 特征提取

&ensp; &ensp; 特征的种类在图像领域主要分为点，线，面。线特征和面特征对图像信息利用得更多，因而其分辨性更高。但遗憾的是，由于线特征和面特征提取的条件比较苛刻，因此在实际应用中并不广泛。（尽管在SLAM中也有点线结合的实例，在图像纹理较弱的情况下，线特征可以发挥更大的用处。但是却是在增加计算量的同时，提高的性能较为有限。）随着深度学习在图像方面的不断提升，基于全图学习得到的特征向量性能不断提高，甚至超越了手工设计的特征点。这也是前面所说的，由于对图像信息利用得更加全面，才使得特征向量的识别性能越来越好。
&ensp; &ensp; 在视觉SLAM中，特征主要还是用点特征。
> 点特征主要分为两大类：
1. **基于手工设计的特征点**：
    这类特征点主要是凭借人们对几何学以及数学上的一些认识，对图像中的某一块特殊区域进行建模得到描述函数。典型的比如SIFT，利用差分高斯金字塔计算图像中比较特殊的点，再通过该点的领域信息对其进行描述，得到最终的描述函数。
2. **基于深度学习得到的特征向量**：
    这类特征点主要是对深度学习网络进行设计，使其通过多个卷积和池化操作来取得图像中比较有用的信息。说起来是比较抽象的，个人理解，深度学习虽然看起来是个大箱子，但是实际上，各个卷基层都是一个函数 $f_{i}(x)$，最终的输出就是多个复合函数的函数值 $G = f_{N}(f_{N-1}(...f_{1}(x)...))$。前期的网络设计主要是模仿手工描述子的计算步骤，各个卷基层的操作与手工描述子并无二致。后期的主要是为了利用更多的图像信息，弥补了手工描述子的不足。（这类网络的设计通常是将手工设计的特征点图块作为训练集，输入到网络中进行训练）。

> 不同点特征的区别：

&ensp; &ensp; 特征点实际上包含两种类型:
1. 一种是只有关键点（interested point)，典型的如FAST这类角点;
2. 还有一类就是基于角点的位置，利用其邻域信息对其进行描述，得到描述子，SLAM中常用的主要是这类描述子，比如SIFT, SURF, ORB等。
   
> 特征提取的缺陷：

&ensp; &ensp; 直接对图像提取特征点，通常会出现特征点扎堆的现象。显然，对于图像纹理多的地方，特征点自然会提取得多。这导致的结果便是，图像某一块位置提取到的特征点特别多，而其他区域提取到的点特别少，甚至是没有。这在实际应用当中会导致估算相对姿态变换出现较大的偏差，影响定位精度。这是我们极力要避免的。

> 特征提取的小技巧：

&ensp; &ensp; 通过对图像进行分块，对不同图像子块提取特征点，可以解决特征点分布不均匀的问题。
&ensp; &ensp; **以ORB-SLAM中提取ORB特征点为例：**
1. 将图像 $I$ 划分成 $4$ 个区域 ${s_1, s_2, s_3, s_4}$，对每个区域提取特征点；
2. 将子图 ${s_1, s_2, s_3, s_4}$ 各自划分成 $4$ 个区域，现在我们总共有 $16$ 个子图，将步骤 $1$ 中提取的特征点按区域进行分配；
3. 对于存在不止一个特征点的子图，进一步进行划分，直到限制的最小子图时停止。若此时特征点数量仍然较多，则根据响应值，取最大的特征点；
4. 若步骤 $1$ 中，各个子图中提取的特征点数量较少，则需要通过调整阈值条件，来提高特征点的数量。
5. 步骤 $1-3$ 可以通过不断递归来完成。


小C酱油兵博客园链接：[特征提取](https://www.cnblogs.com/yepeichu/p/12468228.html).



   