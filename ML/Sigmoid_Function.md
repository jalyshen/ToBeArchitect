# Sigmoid 函数

原文：https://zhuanlan.zhihu.com/p/108641430



## 1. 什么是Sigmoid Function

​        一提起 Sigmoid function可能大家的第一反应就是Logistic Regression。我们把一个sample扔进 $sigmoid$ 中，就可以输出一个probability，也就是是这个sample属于第一类或第二类的概率。

​        还有像神经网络也有用到 $sigmoid$ ，不过在那里叫 activation function（激活函数）。

​        Sigmoid function 形式如下：
$$
\sigma(z) = \frac{1}{1 + e^{-z}}
$$
那么，这个函数是如何来的呢？

## 2. Sigmoid的由来

​        首先假设有两个类别（*Class*) : $C_1$ 和 $C_2$ ，并且给出一个样本Sample $x$，目标是：求 $x$ 属于 $C_1$ 的概率是多少。

​        这个概率可以通过 **Naive Bayes** 很轻松的求得，就是下面（*公式 1*） :
$$
P(C_1 | x) = \frac{P(x|C_1)P(C_1)}{P(x)}
$$
其中，等号右面的分布项 （*公式 2* ）：
$$
P(x) = P(x|C_1)P(C_1) + P(x|C_2)P(C_2)
$$
这个 *公式2* 是高中难度的，简要解释就是： $x$ 出现的概率等于 $C_1$ 出现的概率乘以 $C_1$ 中出现 $x$ 的概率 加上 $C_2$ 出现的概率乘以 $C_2$ 中出现 $x$ 的概率。

​        那么把 *公式2* 带入到 *公式 1* 中，就得到 （*公式 3*）:
$$
P(C_1 | x) = \frac{P(x|C_1)P(C_1)}{P(x|C_1)P(C_1)+P(x|C_2)P(C_2)}
$$
*公式 3* 简化一下（分子分母同时除以分子），得到 （*公式 4*） ：
$$
P(C_1 | x) = \frac{1}{1 + \frac{P(x|C_2)P(C_2)}{P(x|C_1)P(C_1)}}
$$
设：
$$
z = ln\frac{P(x|C_2)P(C_2)}{P(x|C_1)P(C_1)}
$$


把 $z$ 带入 *公式4* ，就得到这个 Sigmoid 函数的现在形式：
$$
\sigma(z) = \frac{1}{1 + e^{-z}}
$$

## 3. 思考

​        已经知道了Sigmoid函数的来龙去脉，那么现在的问题就是：sigmoid函数中，只有 $P(x|C_1)$ 和 $P(x|C_2)$ 不知道，干脆用 **Bayes** 是不是就可以直接计算出 $P(x|C_1)$ 了呢？$x$ 是某个样本，其中有多个feature，也就是说，$x$ 是一个向量（*vector*），那么这个 $P(x|C_1)$ 就是展开就是如下的形式：
$$
P(x|C_1) = P(x_1|C_1)P(x_2|C_1) \ldots P(k_1|C_1) \ldots
$$
但是 **Bayes** 有一个限制条件，就是所有的feature都 **必须是independent** 的，假如训练的sample中各个feature都是independent的话，那么Bayes会给一个很好的结果。但实际情况并非如此完美，各个feature之间不可能是independent的，所以bayes的结果非常大，建立的模型（model ）就不成功。

## 4. 这个$z$ 应该长什么样

​        将 $z$ 变换一下 (根据对数函数特性)，得到下面的形式：
$$
z = ln \frac{P(x|C_1)}{P(x|C_1)} + ln \frac{P(C_1)}{P(C_2)}
$$
上面公式中，$ln \frac{P(C_1)}{P(C_2)}$ 是很好求得的，设 $C_1$ 在训练集中出现的数目是 $N_1$, $C_2$ 在训练集中出现的数目是 $N_2$， 那么有：
$$
ln \frac{P(C_1)}{P(C_2)} = ln \frac{\frac{N_1}{N_1 + N_2}}{\frac{N_2}{N_1 + N_2}}  = ln \frac{N_1}{N_2}
$$
其中， $P(x|C_1)$ 和 $P(x|C_2)$ 都遵从 Guassian Probability Distribution：
$$
P(x|C_1) = \frac{1}{2 \pi^{D/2}} \frac{1}{|\sum_1|^{1/2}} e^{-1/2(x- \mu_1)^T \sum_{1}^{-1}(x - \mu_1)} \\
P(x|C_2) = \frac{1}{2 \pi^{D/2}} \frac{1}{|\sum_2|^{1/2}} e^{-1/2(x- \mu_2)^T \sum_{2}^{-1}(x - \mu_2)}
$$
再回到公式：
$$
z = ln \frac{P(x|C_1)}{P(x|C_1)} + ln \frac{P(C_1)}{P(C_2)}
$$
第二项我们已经求出来了，下面我们把第一项Guassian Probability Distribution带入：
$$
ln \frac{P(C_1)}{P(C_2)} = ln \frac{\frac{1}{2 \pi^{D/2}} \frac{1}{|\sum_1|^{1/2}} e^{-1/2(x- \mu_1)^T \sum_{1}^{-1}(x - \mu_1)}} {\frac{1}{2 \pi^{D/2}} \frac{1}{|\sum_2|^{1/2}} e^{-1/2(x- \mu_2)^T \sum_{2}^{-1}(x - \mu_2)}}
$$
乍一看，简直太复杂太恶心了 :)
但是别慌，很多东西都能消掉的，来消一下。
首先，上面分子分母中 $ \frac{P(C_1)}{P(C_2)}$可以消掉，就变成了：
$$
ln \frac{P(C_1)}{P(C_2)} = ln \frac{\frac{1}{|\sum_1|^{1/2}} e^{-1/2(x- \mu_1)^T \sum_{1}^{-1}(x - \mu_1)}} {\frac{1}{|\sum_2|^{1/2}} e^{-1/2(x- \mu_2)^T \sum_{2}^{-1}(x - \mu_2)}}
$$
然后是：
$$
ln \frac{P(C_1)}{P(C_2)} = ln \frac{|\sum_2|^{1/2}}{|\sum_1|^{1/2}} e^{-1/2[(x- \mu_1)^T \sum_{1}^{-1}(x - \mu_1) - (x- \mu_2)^T \sum_{2}^{-1}(x - \mu_2)]}
$$
接着：
$$
ln \frac{P(C_1)}{P(C_2)} = ln \frac{|\Sigma_{2}|^{1/2}}{|\Sigma_{1}|^{1/2}} - \frac{1}{2}[(x- \mu_1)^T \Sigma_{1}^{-1}(x - \mu_1) - (x- \mu_2)^T (\Sigma_{2})^{-1}(x - \mu_2)]
$$


上面第二项 $\frac{1}{2}[(x- \mu_1)^T (\Sigma_{1})^{-1}(x - \mu_1) - (x- \mu_2)^T (\Sigma_{2})^{-1}(x - \mu_2)]$ ，中括号里面有两项，再把这两项里面的括号全部打开，打开的目的是为了后面的简化。

​        先来看第一项：
$$
(x - \mu_1)^T(\Sigma_{1})^{-1}(x - \mu_1) 
= x^T \Sigma_{1}^{-1}x - x^T \Sigma_1^{-1} \mu_1 - \mu_1^{T} \Sigma_1^{-1} x + \mu_1^{T} \Sigma^{-1} \mu_1 \\
= x^T \Sigma_{1}^{-1}x -2 \mu_1^{T} \Sigma_1^{-1} x + \mu_1^{T} \Sigma_1^{-1} \mu_1
$$
​        第二项化简方法一样，把下角标换成2就行了：
$$
(x - \mu_2)^T(\Sigma_2)^{-1}(x - \mu_2)  \\
= x^T \Sigma_{2}^{-1}x -2 \mu_2^{T} \Sigma_2^{-1} x + \mu_2^{T} \Sigma_2^{-1} \mu_2
$$
拆得差不多了，回到 $z = ln \frac{P(x|C_1)}{P(x|C_2)} + ln \frac{P(C_1)}{P(C_2)}$ 中，把刚才化简结果带进去：
$$
z = ln \frac{|\Sigma_{2}|^{1/2}}{|\Sigma_{1}|^{1/2}} - \frac{1}{2}[x^T \Sigma_{1}^{-1}x -2 \mu_1^{T} \Sigma_1^{-1} x + \mu_1^{T} \Sigma_1^{-1} \mu_1 - x^T \Sigma_{2}^{-1}x +2 \mu_2^{T} \Sigma_2^{-1} x - \mu_2^{T} \Sigma_2^{-1} \mu_2] + ln \frac{N_1}{N_2}
$$
仔细观察不难发现，上式中中括号里面第一项和第四项是可以消掉的。

​        这里，可以认为 $\Sigma_1 = \Sigma_2 = \Sigma$ ，刚才一直没有解释 $\mu$ 和 $ \Sigma$ 是什么，现在简单说一下： $\mu$ 就是mean (均值) ，$\Sigma$ 就是 covairance （协方差），其中 $\mu$ 是向量 (Vector) ，$\Sigma$ 是个Matrix，具体什么形式，可以参考 Guassian 看看paper。

​        那么，为什么可以认为 $\Sigma_1 = \Sigma_2 = \Sigma$  呢？因为如果每个class（分类）都有自己的Covariance的话，那么Variance 会很大，参数的量一下子就上去了，参数一多，就容易 overfitting。 这么说的话， $z$ 里面的第一项 $ln = \frac{|\Sigma_{2}|^{1/2}}{|\Sigma_{1}|^{1/2}}$ 就是 $0$了。这样，就又可以简化公示了。

​        最后， $z$ 就可以被简化成了这个最终形态了：
$$
z = (\mu_1 - \mu_2) \Sigma^{-1} x - \frac{1}{2} \mu_1^{T} \Sigma^{-1} \mu_1 + \frac{1}{2} \mu_2^{T} \Sigma^{-1} \mu_2 + ln \frac{N_1}{N_2}
$$
可以观察到，第一项有系数 $x$，后面几项其实都是参数。这就可以理解为 $x$ 的系数其实就是 $sigmoid$ 中的参数 $\omega ^T$ (这是个matrix)，后面那些项可以看成是参数 $b$ 。

​        在Generative Model 中，目标是寻找最佳的 $N_1, N_2, \mu_1, \mu_2, \Sigma$ 使得 $P(C_1 |x)$ 最大化。但已经将一连串复杂的参数和方程简化成了 $z = \sigma(\omega^{T}x + b)$ ，那为什么还要舍近求远的求 5 个参数去目标最优化呢？只有“**两个参数**”的方法叫做***Discraminative Model*** 。

​        实际上，在大多数情况下，这两种方法各有利弊，但是实际上 Discraminative Model 泛化能力比 Generative Model 还是强很多。那么，什么时候 Generative Model更好呢？这两种情况：

* Training Data比较少的时候，需要靠几率模型脑补没有发生过的事情
* Training Data中有noise