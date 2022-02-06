# 交叉熵损失函数(Cross Entropy Error Function)与均方差损失函数(Mean Squared Error)

原文：https://blog.csdn.net/bitcarmanlee/article/details/105619286



文中涉及到的 sigmoid 函数，是一种**激活函数**，[这里](./Sigmoid_Function.md)对其有介绍。

## 1. 均方差损失函数 (Mean Squared Error)

均方差损失函数是预测数据和原始数据对应点误差的平方和的均值。计算方式比较简单：
$$
MSE = \frac{1}{N}(\hat{y} - y)^2
$$
其中：

* $N$ 为样本个数
* $\hat{y}$ 预测数据值
* $y$ 原始数据值

## 2. 交叉熵损失函数 (Cross Entropy Error Function)

在分类问题中，尤其是在神经网络中，交叉熵函数非常常见。因为经常涉及到分类问题，需要计算各类别的概率，所以交叉熵损失函数又都是与 sigmoid 函数或者 softmax 函数成对出现。

比如用神经网络最后一层作为概率输出，一般最后一层神经网络的计算方式如下：

1. 网络的最后一层得到每个类别的scrores
2. socre与sigmoid函数或者softmax函数进行计算得到概率输出
3. 第二步得到的类别概率与真实类别的one-hot形式进行交叉熵计算

**二分类**的交叉熵损失函数形式如下：
$$
\sum-y_i log(\hat{y_i}) - (1 - y_i) log(1 - \hat{y_i})
$$
其中：

* $y_i$ 表示类别为 $1$
* $\hat{y_i}$ 表示预测类别为 
* $1$ 的概率

**多类别**的交叉熵损失函数形式为：
$$
-\sum_{i=1}^{n}y_ilog(\hat{y_i})
$$
这个式子表示，有 $n$ 个类别。单分类问题的时候，$n$ 个类别是 one-hot 的形式，只有一个类别 $y_i = 1$ ，其他 $n-1$ 个类别分别为 $0$  。

## 3. MSE 与 Sigmoid 函数不适合配合使用

MSE的形式如上面所示，那么MSE的loss则为：
$$
Loss_{mse} = - MSN = - \frac{1}{N}(\hat{y} - y )
$$
如果其与sigmoid函数配合使用，偏导数为：
$$
\frac{\partial Loss_i}{\partial \omega} = (y - \hat{y})\sigma' (\omega x_i + b) x_i
$$
其中：
$$
\sigma'(\omega x_i + b)  = \sigma (\omega x_i + b)(1 - \sigma(\omega x_i + b))
$$
于是，在 $\sigma (\omega x_i + b)$ 的值接近 $0$ 或者 $1$ 的时候，其导数都接近 $0$ 。这样会导致模型一开始的学习速度非常慢，所以 MSE 一般不会与 sigmoid 函数配合使用。

## 4. 交叉熵损失函数与 Sigmoid 函数配合使用

交叉熵损失函数与 Sigmoid 函数配合使用，最终损失函数求导的结果为：
$$
\frac{\partial Loss_i}{\partial \omega} = ( \hat{y} - y_i) x_i
$$
由此可见，求导的结果与 $\hat{y} - y_i$ 与 $x_i$ 的值有关，不会出现模型开始训练速度很慢的现象。



## 5. 交叉熵损失函数与 softmax 函数配合使用

在神经网络中，交叉熵损失函数经常与Softmax配合使用。交叉熵的损失函数如下：
$$
Loss = - \sum_i t_i ln{y_i}
$$
Softmax 函数：
$$
y_i = \frac{e^i}{\sum_j{e^j}} = 1 - \frac{\sum_{j \neq i}{e^j}}{\sum_j{e^j}}
$$
接下来求导：
$$
\begin{align*}
\frac{\partial Loss_i}{\partial_i} &= - \frac{ \partial ln{y_i}}{ \partial_i} \\
&= - \frac{\sum_j{e^j}}{e^j} \cdot \frac{\partial(\frac{e_j}{\sum_j{e^j}})}{\partial_i} \\
&= - \frac{\sum_j{e^j}}{e^j} \cdot (- \sum_{j \neq i}{e^j}) \cdot \frac{\partial(\frac{1}{\sum_j{e^j}})}{\partial_i} \\
&= \frac{\sum_j{e^j} \cdot \sum_{j \neq i}{e^j}}{e^j} \cdot \frac{-e^j}{(\sum_j{e^j})^2} \\
&= - \frac{\sum_{j \neq i}{e^j}}{\sum_j{e^j}} \\
&= - (1 - \frac{e^j}{\sum_j{e^j}}) \\
&= y_i - 1
\end{align*}
$$
由此可见，交叉熵函数与softmax函数配合，损失函数求导非常简单！

