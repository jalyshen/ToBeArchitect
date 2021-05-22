# 欧氏距离 (Euclidean Distance)

原文：https://blog.csdn.net/weixin_39955953/article/details/111887157



欧几里得度量(欧氏距离)是最易于理解的一种距离计算方法，源自欧氏空间中两点间的距离公式。



### 1. 平面上两点的欧氏距离

二维平面 $\mathbb{R^2}$ 的上两个点$A(x_1,y_1)$和$B(x_2,y_2)$ 的距离计算公式：
$$
D = \sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^2}
$$
特殊的，如果其中一个点是原点，那么平面内一个点 $A(x,y)$ 到原点的欧式距离可以简化为：
$$
D = \sqrt{x^2 + y^2}
$$


### 2. 空间中两点的欧氏距离

三维空间 $\mathbb{R^3}$ 中，两个点 $A(x_1,y_1,z_1)$ 和 $B(x_2,y_2,z_2)$ 的距离计算公式：
$$
D = \sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^2 + (z_1 - z_2)^2}
$$
同样的，如果其中一个点是原点，那么三维空间内一个点 $A(x,y,z)$ 到原点的欧式距离可以简化为：
$$
D = \sqrt{x^2 + y^2 + z^2}
$$


> ***以上两个距离公式，均可以很轻松的通过勾股定理证明***



### 3. $N$ 维空间 $\mathbb{R^n}$ 中两点的欧氏距离

当空间维度超过3，无法直观的看到两个点的距离，可以根据欧氏距离在平面和三维空间的计算原理，得到N
维空间 $\mathbb{R^n}$ 中，两点 $A=(a_1,a_2,\cdots,a_n)$ 和 $B=(b_1,b_2,\cdots,b_n)$ 的距离计算公式：
$$
D = \sqrt{(a_1 - b_1)^2 + (a_2 - b_2)^2 + (a_3 - b_3)^2 + \cdots + (a_n - b_n)^2}
$$
采用数学符号，可以缩写成：
$$
D = \sqrt{ \sum_{i=1}^{n}(a_i - b_i)^2}
$$
按照两个**向量**相乘的方式写，就是：
$$
D = \sqrt{(A - B) \cdot (A - B)^T}
$$