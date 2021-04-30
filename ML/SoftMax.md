# SoftMax函数介绍

原文： https://blog.csdn.net/bitcarmanlee/article/details/82320853



## SoftMax函数简介

​        在机器学习中，尤其是深度学习，softmax是一个非常常用而且比较重要的函数，尤其在多分类的场景中使用广泛。他把一些输入映射为0-1之间的实数，并且归一化保证和为1。因此多分类的概率之和也刚好为1。

​        首先简单来看看softmax是什么意思。顾名思义，softmax由两个单词构成，soft和max。

​        先看max。对于max我们都很熟悉，比如有两个变量a、b。如果a > b，则max(a, b) = a， 反之则 max(a, b) = b。另一个词是soft。如果将max看成一个分类问题，就是非黑即白，最后的输出是一个确定的变量。更多时候，我们希望输出的是取到某个分类的概率，或者说，希望分值大的那一项被**经常取到**，而分值较小的那一项也有一定概率被取到，所以这就是应用了soft的概念。即最后的输出是每个分类被取到的概率。

## SoftMax定义

​        首先看一个图，图中清晰的描述了softmax是怎么计算的：

![softmax-theory](./images/softmax/softmax-theory.jpg)

​        假设有一个数组$V$，$V_i$ 表示 $V$ 中的第 $i$ 个元素，那么这个元素的softmax的值为：
$$
S_i = \frac{e^i}{\sum_j{e^j}}
$$
该元素的softmax值，就是该元素的指数与所有元素指数和的比值。

​        这个定义可以说非常简单，也很直观。那么为什么要定义成这个形式呢？原因如下：

1. softmax设计的初衷，是希望特征对概率的影响是乘性的
2. 多类分类问题的目标函数常常选为cross- entropy，即 $L = - \sum_k{t_k} \cdot lnP(y =k)$ , 其中目标类的 $t_k$ 为$1$ ，其余类的 $t_k$ 为 $0$

​        在神经网络模型中（最简单的logistic regression也可以看成没有隐含层的神经网络），输出层第 $i$ 个神经元的输入为 $\alpha_i =\sum_d{\omega}_{id}x_d$ 

​        神经网络使用error back-propagation训练的，这个过程中有一个关键的量是$\delta L/\delta \alpha_i$ 。



## Softmax求导

​        前面提到，在多分类问题中，经常使用交叉熵作为损失函数：
$$
Loss = - \sum_i{t_i ln{y_i}}
$$
其中， $t_i$ 表示真实值,  $y_i$ 表示求出的softmax的值。当预测第 $i$ 个时， 可以认为 $t_i = 1$ 。此时损失函数变成了：
$$
Loss_i = - ln{y_i}
$$
接下来对 $Loss$ 求导。根据定义：
$$
y_i = \frac{e^i}{\sum_j{e^j}}
$$
已经将数值映射到了 0-1 之间，并且和为 1， 则有：
$$
\frac{e^i}{\sum_j{e^j}} = 1 - \frac{\sum_{j \neq i}{e^j}}{\sum_j{e^j}}
$$
接下来求导：
$$
\frac{\delta Loss_i}{\delta_i} = - \frac{\delta ln {y_i}}{\delta_i} \\
= \frac{\delta (- ln{\frac{e^i}{\sum_j{e^j}}})}{\delta_i} \\
= - \frac{1}{\frac{e^i}{\sum_j}{e^j}} \cdot \frac{\delta (\frac{e^i}{\sum_j{e^j}})}{\delta_i} \\
= - \frac{\sum_j{e^j}}{e^i} \cdot \frac{\delta (1 - \frac{\sum_{j \neq i}e^j}{\sum_j{e^j}})}{\delta_i} \\
= - \frac{\sum_j{e^j}}{e^i} \cdot (-\sum_{j \neq i}e^j) \cdot \frac{\delta (\frac{1}{\sum_j{e^j}})}{\delta_i} \\
= \frac{\sum_j{e^j} \cdot \sum_{j \neq i}{e^j}}{e^i} \cdot \frac{-e^i}{(\sum_j{e^j})^2} \\
= - \frac{\sum_{j \neq i}{e^j}}{\sum_j{e^j}} \\
= - (1 - \frac{e^i}{\sum_j{e^j}}) \\
= y_i - 1
$$
上面的结果表示，我们只需要正想求出 $y_i$ ，将结果减1就是反向更新的梯度，导数的计算是不是非常简单！

上面的推导过程会稍微麻烦一些，特意整理了一下，结合交叉熵损失函数，整理了一篇新的内容，看起来更直观一些：

[交叉熵损失函数(Cross Entropy Error Function)与均方差损失函数(Mean Squared Error)](./CrossEntropy_and_MeanSquared.md)



## Softmax *vs* K个二元分类器

​        如果你在开发一个音乐分类的应用，需要对k种类型的音乐进行识别，那么是选择使用 softmax 分类器呢，还是使用 logistic 回归算法建立 k 个独立的二元分类器呢？
​        这一选择取决于你的类别之间是否互斥，例如，如果你有四个类别的音乐，分别为：古典音乐、乡村音乐、摇滚乐和爵士乐，那么你可以假设每个训练样本只会被打上一个标签（即：一首歌只能属于这四种音乐类型的其中一种），此时你应该使用类别数 k = 4 的softmax回归。（如果在你的数据集中，有的歌曲不属于以上四类的其中任何一类，那么你可以添加一个“其他类”，并将类别数 k 设为5。）
​        如果你的四个类别如下：人声音乐、舞曲、影视原声、流行歌曲，那么这些类别之间并不是互斥的。例如：一首歌曲可以来源于影视原声，同时也包含人声 。这种情况下，使用4个二分类的 logistic 回归分类器更为合适。这样，对于每个新的音乐作品 ，我们的算法可以分别判断它是否属于各个类别。
​        现在我们来看一个计算视觉领域的例子，你的任务是将图像分到三个不同类别中。(i) 假设这三个类别分别是：室内场景、户外城区场景、户外荒野场景。你会使用sofmax回归还是 3个logistic 回归分类器呢？ (ii) 现在假设这三个类别分别是室内场景、黑白图片、包含人物的图片，你又会选择 softmax 回归还是多个 logistic 回归分类器呢？
​        在第一个例子中，三个类别是互斥的，因此更适于选择softmax回归分类器 。而在第二个例子中，建立三个独立的 logistic回归分类器更加合适。