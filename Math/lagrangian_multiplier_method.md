# 拉格朗日乘子法

原文：https://blog.csdn.net/qq_33829547/article/details/100152556



## 引言

​        学习支持向量机的原理时，会遇到**带约束条件的机制问题**。那么理解拉格朗日乘子法就很有必要了。

## 简单案例

### 问题1

​     设有一条绳子，长4米，将绳子围城长为 $x$ ，宽为 $y$ 的矩形，求矩形的最大面积是多少？

####     求解

​        估计大家很容易列出如下的方程组，然后进行**消元**，然后求出极值：
$$
\begin{aligned}
\left\{\begin{matrix}
  S &=& xy \\
  2(x + y) &=& 4
\end{matrix}\right.
\end{aligned}
$$
​        消元：
$$
y = 2 - x
$$
​        代入：
$$
S = 2x - x^2
$$
​        求极值：
$$
S' =2 - 2x = 0
$$
​        解得$x = 1, y = 1$，也就是当长为1米，宽为1米时矩形面积最大。



​        有人会问，这也没见到拉格朗日乘子法啊？没错，如上面这般简单的条件极值问题确实不需要拉格朗日乘子法，但如果问题一旦复杂起来，比如下面的问题。

### 问题2

​        一个长方体的长宽高分别是 $x,y,z$，表面积是 $a^2$，问当长宽高具体为多少时，体积最大？

​       按照上面消元代入法的步骤，你会得到如下的目标函数：
$$
V = \frac{xy}{2}(\frac{a^2 - 2xy}{x + y})
$$
​       **虽然也把条件极值问题成功转化为无条件极值问题了，但是求极值就麻烦多了**，涉及多元函数求极值。如果问题进一步复杂，比如**有很多个自变量，有很多个约束条件**，那将更加爆炸。

​       时机到了，请出我们今天的主角——**拉格朗日乘子法**。



## 拉格朗日乘子法

​        有函数 $f ( x , y )$，要找在约束条件 $ \psi(x,y)=0$ 下的极值点。我们先构造**拉格朗日函数**：
$$
F(x,y,\lambda) = f(x,y) + \lambda \psi(x,y)
$$
然后分别对 $x, y, \lambda$ 求偏导，得到方程组：
$$
\begin{aligned}
\left\{\begin{matrix}
  \frac{\partial{F(x,y,\lambda)}}{\partial{x}} = 0 \\
  \frac{\partial{F(x,y,\lambda)}}{\partial{y}} = 0 \\
  \frac{\partial{F(x,y,\lambda)}}{\partial{\lambda}} = 0
\end{matrix}\right.
\end{aligned}
$$
可以求得 $x, y$，代回原函数就能得出极值。

​        拉格朗日乘子法也可推广到**多元变量**和**多约束条件**的情况，如：

​        要找函数 $f(x,y,z,t) $ 在约束条件 $ \psi(x,y,z,t)=0$ 和 $ \Psi(x,y,z,t)=0 $ 下的极值。构造拉格朗日函数：
$$
F(x,y,z,t,\lambda_1, \lambda_2) = f(x,y,z,t) + \lambda_1 \psi(x,y,z,t) + \lambda_2 
\Psi(x,y,z,t)
$$
分别求偏导，建立方程组：
$$
\begin{aligned}
\left\{\begin{matrix}
  \frac{\partial{F(x,y,z,t,\lambda_1,\lambda_2)}}{\partial{x}} = 0 \\
  \frac{\partial{F(x,y,z,t,\lambda_1,\lambda_2)}}{\partial{y}} = 0 \\
  \frac{\partial{F(x,y,z,t,\lambda_1,\lambda_2)}}{\partial{z}} = 0 \\
  \frac{\partial{F(x,y,z,t,\lambda_1,\lambda_2)}}{\partial{t}} = 0 \\
  \frac{\partial{F(x,y,z,t,\lambda_1,\lambda_2)}}{\partial{\lambda_1}} = 0 \\
  \frac{\partial{F(x,y,z,t,\lambda_1,\lambda_2)}}{\partial{\lambda_2}} = 0 \\
\end{matrix}\right.
\end{aligned}
$$
解出 $x、y、z、t$，代回原函数即可得到极值。



## 具体应用

​        如果只放公式似乎太抽象了，我在这里就在具体是案例中使用拉格朗日乘子法计算极值。

​        还是用上面的问题2：

​        一个长方体的长宽高分别是$x,y,z$，表面积是 $a^2$ ，问当长宽高具体为多少时，体积最大？

体积函数：
$$
V = xyz
$$
**约束条件**(根据面试得出式子)：
$$
2xy + 2xz + 2yz - a^2 = 0
$$
**构造拉格朗日函数**：
$$
F(x,y,z,\lambda) = xyz + \lambda(2xy + 2xz + 2yz - a^2)
$$
分别求导，建立方程组：
$$
\begin{aligned}
\left\{\begin{matrix}
  \frac{\partial{F(x,y,z,\lambda)}}{\partial{x}} &=& yz + \lambda(2y + 2z) =0 \\
  \frac{\partial{F(x,y,z,\lambda)}}{\partial{y}} &=& yz + \lambda(2y + 2z) =0 \\
  \frac{\partial{F(x,y,z,\lambda)}}{\partial{z}} &=& yz + \lambda(2y + 2z) =0 \\
  \frac{\partial{F(x,y,z,\lambda)}}{\partial{\lambda}} &=& 2xy + 2xz + 2yz - a^2 = 0 \\
\end{matrix}\right.
\end{aligned}
$$
求解得：
$$
x = y = z = \frac{a}{\sqrt{6}}
$$
代入体积函数，可得最大体积：
$$
V_{max} = \frac{\sqrt{6}a^3}{36}
$$
