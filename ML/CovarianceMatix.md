# 透彻理解协方差矩阵

原文：https://www.toutiao.com/a6640622226158649860/?channel=&source=search_tab



​        协方差及协方差矩阵有着特别广泛的应用，在多元高斯分布、高斯过程、卡尔曼滤波等算法中多有用到，本文从协方差、协方差矩阵讲起，并重点讲解协方差矩阵在高斯分布中的用法及意义，也是讲解高斯过程、贝叶斯优化的铺垫。



## 一. 协方差(Covariance)

​        $X, \ Y$ 两个随机变量的协方差在和中用于衡量两个变量的总体。<font color='red'>**用来刻画两个随机变量之间的相关性** </font>：
$$
\operatorname{Cov}(X, Y) \equiv C_{XY} \equiv E((X- \mu_x)(Y - \mu_y))
$$
*这里 $E(x) = \mu$ 表示期望函数值*。假定，不知道潜在的概率分布，可以取 $n$ 个样本来计算：
$$
\operatorname{Cov}(X, Y) = \frac{1}{n} \sum_{i=1}^{n} (x_i - m_x)(y_i - m_y)
$$
分别计算这 $n$ 样本的两个变量的均值，这两个变量的协方差可以用下式来计算：
$$
\operatorname{Cov}(x,y) = \frac{1}{n} [x_1 - m_x \ \ x_2 - m_x \ \ \cdots \ \ x_n - m_x] \begin{bmatrix}
 y_1 - m_y\\
 y_2 - m_y\\
 \vdots \\
y_n - m_y
\end{bmatrix}
$$
由于变量都有**量纲**，如果消除各自量纲影响，将协方差除以两个变量的**标准差**，则可得相关系数：
$$
\rho_{xy} \equiv \frac{E\left ((X - \mu_x)(Y - \mu_y) \right)}{\sigma_x \sigma_y } = \frac{Cov(X,Y)}{\sigma_x \sigma_y}
$$

其中，$\sigma_x, \sigma_y$ 分别是向量 $X, Y$ 的标准差。注：$\sigma_x = \sqrt{\frac{\sum_{i=1}^{n}(x-\bar{x})^2}{n-1}}$

## 二. 协方差矩阵

随机向量：
$$
X = (x_1, x_2, \cdots , x_n)^T
$$
计算所有元素的两两协方差，形成协方差矩阵：
$$
\sum_X \equiv \begin{pmatrix}
  C_{11}&  C_{21}&  \cdots& C_{n1}\\
  C_{12}&  C_{22}&  \cdots& C_{n2}\\
  \vdots&  \vdots&  \ddots& \vdots\\
  C_{1n}&  C_{2n}&  \cdots& C_{nn}
\end{pmatrix} \\
\ \\
\operatorname{Cov}(x_i, x_j) = c_{ij} = E((x - \mu_i)(x_j - \mu_j))
$$
这是一个对称矩阵，对角线是每个变量的方差。如果是对角阵，协方差矩阵形式如下：
$$
\sum_X \equiv \begin{pmatrix} \\
\sigma_1^2& 0& \cdots& 0 \\
0& \sigma_2^2& \cdots& 0 \\
\vdots& \vdots& \ddots& \vdots \\
0& 0& \cdots& \sigma_n^2 \\
\end{pmatrix} \\
$$

## 三. 协方差矩阵与多元高斯分布

**多元高斯分布概率密度的推导**

设多元高斯分布如下：均值向量为 $\mu$ , 协方差矩阵为$\Sigma$， 则有：
$$
X \sim N(\mu, \Sigma) \\
p(X) = \frac{1}{(2\pi)^{D/2}|\Sigma|^{1/2}} e^{-\frac{1}{2}(X-\mu)^T \Sigma_{}^{-1} (X - \mu)}
$$
与一元高斯分布对比，概率密度函数形式有所变化，这个变化是怎么来的呢？通过二元高斯分布来推导一下。

对于二元高斯分布，设定：
$$
x = \begin{bmatrix}
 x_1\\
 x_2
\end{bmatrix} \ \ \ \ \ \ \  
\mu = \begin{bmatrix}
 \mu_1\\
 \mu_2
\end{bmatrix} \ \ \ \ \ \ \  
\Sigma = \begin{bmatrix}
  \sigma_1^2& 0 \\
  0& \sigma_2^2
\end{bmatrix}
$$
现在推导两个变量的高斯分布的密度函数公式：
$$

$$
