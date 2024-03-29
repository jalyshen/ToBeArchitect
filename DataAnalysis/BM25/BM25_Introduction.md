# BM25详解

原文一：https://blog.csdn.net/wdxin1322/article/details/38093261

原文二：https://blog.csdn.net/wang_shen_tao/article/details/50682478

## 什么是BM25

**BM25算法通过加入文档权值和查询权值，扩展了二元独立模型的得分函数**。这种拓展是基于概率论和实验验证的，并不是一个正式的模型。BM25模型在二元独立模型的基础上，考虑了单词在查询中的权值，拟合综合上述考虑的公式，并通过实验引入经验参数。

BM25的原始公式：

![](./images/BM25_original.png)



Log后由三部分组成，其中第一部分是**二元独立模型的计算得分**；第二部分是**文档权值**；第三部分是**查询权值**。

要了解BM25模型，先要了解二元独立模型（BIM，binary independence model）。

## 二元独立模型(BIM)

BIM(binary independence model) 是为了对文档和 query （查询）相关性评价而提出的算法，BIM 为了计算 $P(R|d,q)$（P：概率；R：相关性；d，文档权值；q，查询权值），引入了两个基本假设：

**假设一：二元假设**

所谓二元假设，类似布尔二模型的表示方法。一篇文章在**由特征表示**的时候，以特征“**出现**”和**“不出现**”两种情况表示，也可以理解为相关不相关。用$$P(D|R)$$表示。*注：公式中的$$R$$，表示“相关性” ； $D$ 表示文档*。

例如：文档 d 在表示为向量 $\vec x=(x_1,x_2,...,x_n)$，其中当词 $t$ 出现在文档 d 时，$x_t=1$，否在 $x_t=0$。

**假设二：词汇独立性假设**

所谓独立性假设，是指文档里出现的单词之间没有任何关联，任意一个单词在文章中的分布率不依赖于另一个单词是否出现，这个假设明显与事实不符，但是为了简化计算，很多地方需要做出独立性假设，这种假设是普遍的。用$$P(D|NR)$$表示。*注：公式中的$$NR$$，表示“不相关性” ； $D$ 表示文档。*



在以上两个假设的前提下，二元独立模型即可以对两个因子$$P(D|R)$$和$$P(D|NR)$$进行估算（条件概率）。举个简单例子：文档D的特征向量由五个单词构成 $\vec x=(x_1,x_2,x_3,x_4,x_5)$，每个特征值出现情况如下：$\vec x=(1, 0, 1, 0, 1)$，其中0表示不出现，1表示出现。

用 $$P_i$$ 表示第 $$i$$ 个单词在相关文档中出现的概率，在**已知相关文档集合**的情况下，观察到文档D的概率为：
$$
P(D|R) = p_1 \times (1 - p_2) \times p_3 \times (1 - p_4) \times p_5
$$
对于因子$$P(D|NR)$$，我们假设用 $$S_i$$ 表示第 $$i$$ 个单词在在**不相关文档集合**中出现的概率，于是在已知不相关文档集合的情况下，观察到文档D的概率为：
$$
P(D|NR) = s_1 \times (1-s_2) \times s_3 \times (1-s_4) \times s_5
$$
于是就可以的到下面的估算：
$$
\frac{P(D|R)}{P(D|NR)} = \frac{p_1 \times (1 - p_2) \times p_3 \times (1 - p_4) \times p_5}{s_1 \times (1-s_2) \times s_3 \times (1-s_4) \times s_5}
$$
可以将各个因子规划为两个部分：一部分是文档D中出现的各个单词的概率乘积，另一部分是没在文档D中出现的各个单词的概率乘积，于是公式可以理解为下面的形式：
$$
\frac{P(D|R)}{P(D|NR)} = \prod_{i:d_i=1}^{}\frac{p_i}{s_i} \times \prod_{i:d_i=0}^{}\frac{1-p_i}{1-s_i}
$$
进一步对公式进行变化，可得：
$$
\begin{aligned}
\frac{P(\mathrm{D} \mid \mathrm{R})}{P(\mathrm{D} \mid \mathrm{NR})}= &\prod_{i: d_{i}=1} \frac{p_{i}}{s_{i}} \times\left(\prod_{i: d_{i}=1} \frac{1-s_{i}}{1-p_{i}} \times \prod_{i: d_{i}=1} \frac{1-p_{i}}{1-s_{i}}\right) \times \prod_{i: d_{i}=0} \frac{1-p_{i}}{1-s_{i}} \\
=&\left(\prod_{i: d_{i}=1} \frac{p_{i}}{s_{i}} \times \prod_{i: d_{i}=1} \frac{1-s_{i}}{1-p_{i}}\right) \times\left(\prod_{i: d_{i}=1} \frac{1-p_{i}}{1-s_{i}} \times \prod_{i: d_{i}=0} \frac{1-p_{i}}{1-s_{i}}\right) \\
=&\prod_{i: d_{i}=1} \frac{p_{i}\left(1-s_{i}\right)}{s_{i}\left(1-p_{i}\right)} \times \prod \frac{1-p_{i}}{1-s_{i}}
\end{aligned}
$$


第一部分代表在文章中**特征词**出现过所计算得到的单词概率乘积，第二部分表示**所有词**计算得到单词概率乘积，它与具体的文档无关，在排序中不起作用，可以抹掉。最后得到估算公式：
$$
\frac{P(\mathrm{D} \mid \mathrm{R})}{P(\mathrm{D} \mid \mathrm{NR})}=\prod_{i: d_{i}=1} \frac{p_{i}(1-s_{i})}{s_{i}(1-s_{i})}
$$
为了方便计算，对上面的公式两边取对数([为什么要取对数](https://zhuanlan.zhihu.com/p/106232513))得到：
$$
\sum \log \left(\frac{P i \cdot(1-S i)}{\operatorname{Si} \cdot(1-P i)}(i: d i=1)\right.
$$
或者这样的形式：
$$
\sum_{i: d_{i}=1} \log \frac{p_{i}\left(1-s_{i}\right)}{s_{i}\left(1-p_{i}\right)}
$$
那么如何估算概率 $$s_{i}$$ 和 $$p_{i}$$ 呢？如果给定用户查询，能确定哪些文档集合构成了相关文档集合，哪些文档构成了不相关文档集合，那么就可以用如下的数据对概率进行估算：

![](./images/Si-Pi-Table.jpg)

> 关键词解释：
>
> * $N$: 表示是文档集中总的文档数
> * $R$: 表示与query相关的文档数
> * $r_i$: 表示与query相关的文档中含有的第i个term文档个数
> * $n_i$: 表示含有的第 $i$ 个term文档总数
> * $0.5$: 是平滑因子，避免出现$log(0)$

根据上表，可以计算出$$s_{i}$$ 和 $$p_{i}$$ 的概率估值，为了避免$$log(0)$$，对估值公式进行平滑操作，分子 $+0.5$， 分母 $+1.0$：
$$
\begin{align}
p_{i} =& \frac{(r_{i} + 0.5)}{ (R + 1.0) }\\
s_{i} =& \frac{(n_{i} - r_{i} + 0.5)} {(N - R + 1.0)}
\end{align}
$$
代入估值公式，得到：
$$
\sum_{i: q_{i}=d_{i}=1} \log \frac{\left(r_{i}+0.5\right) /\left(R-r_{i}+0.5\right)}{\left(n_{i}-r_{i}+0.5\right) /\left((N-R)-\left(n_{i}-r_{i}\right)+0.5\right)}
$$

这个公式代表的含义是：对于同时出现在查询Q和文档D中的单词，**累加**每个单词的估值结果就是***: 文档D和查询Q的相关性度量***。

在预先不知道哪些文档相关哪些文档不相关的情况下，可以使用固定值替代，这种情况下，该公式等价于向量空间模型（VSM）中的IDF因子，实际证明该模型的实际使用结果不好，但是它是BM25模型的基础。

​        

## BM25模型详解

BIM(二元假设模型)对于单词特征，只考虑单词是否在文档D中出现过，并不考虑单词本身的相关特征。

BM25在BIM的基础上引入**单词在查询中的权值**，单词在文档D中的权值，以及一些经验参数，所以BM25在实际应用中效果远远好于BIM模型。

在开篇，已经介绍了BM25公式，也说明该公式由三部分组成。下图就是对该公式的分解：

![](./images/BM25.jpg)

* 第一部分是BIM模型得分。上面提到了，在一定的情况下该部分等价于IDF

* 第二部分是查询词在文档D中的权值，$f_i$ 是查询词在文档中的频率，$$k_1$$ 和 $$K$$ 是***经验参数***

* 第三部分是查询词自身的特征，$qf_i$ 是查询词在用户查询中的频率，但一般用户查询都比较短，$qf_i$ 一般是 $1$， $$k_2$$是***经验参数***

从上面的公式可以看出BM25是查询中单词的分值叠加得到，每个单词是一个个体，而整个文档被作为一个整体。

在第二部分中，$$K$$ **因子代表了文档长度的考虑**，$$dl$$ 是文档的长度，$$avdl$$ 是文档的平均长度，$$k_1$$和$$b$$是调整参数，$$b=0$$ 时即不考虑文档长度的影响，**经验表明，$$b=0.75$$ 左右效果比较好**。但是也要根据相应的场景进行调整。**$$b$$ 越大对文档长度的惩罚越大**。$$k_1$$ 因子用于调整词频，极限情况下$$k_1=0$$，则第二部分退化成 $$1$$，即词频特征失效，可以证明 **$$k1$$ 越大词频的作用越大**。

在不知道哪些文档相关、哪些文档不相关的情况下，将文档相关数$R$ 及包含查询相关文档数 $$r$$设为 $$0$$，那么第一部分的BIM公式退化成：

![](./images/BIM-Simple.jpg)

这就是IDF因子的定义，$$N$$ 是总文档数，$$n$$ 是查询词的tf信息，$$0.5$$ 是平滑因子。

以上就是BM25的完整定义及其解释了。



## BM25F

BM25在计算相关性时，把文档当作整体来考虑，但是随着搜索技术的发展，文档慢慢的被结构化数据所替代（比如非结构化数据被存储在了Hive，HBase等存储里），每个文档都会被切分成多个独立的域，尤其是垂直化的搜索。

例如，网也有可能被切分成成标题、内容、主题词等域，这些域对文章主题的贡献不能同等对待，所有权终究要有所偏重，BM25没有考虑这点。

BM25F就是在此基础上做了一些改进，就是不再单单地将单词作为个体考虑，而是将文档也按照field（field是分词术语，也是BM25F名称的由来）划分为个体再考虑，所以BM25F是每个单词在各个field中分值的权求和。

$$
\operatorname{weight}(t, d)=\sum_{\operatorname{c \text { in }} d} \frac{\operatorname{occurs}_{t, c}^{d} \cdot b o o s t_{c}}{\left(\left(1-b_{c}\right)+b_{c} \cdot \frac{l_{c}}{a v l_{c}}\right)}
$$

$$
\begin{array}{c}
R(q, d)=\sum_{\operatorname{t \text { in }} q} \frac{\text { weight }(t, d)}{k_{1}+\text { weight }(t, d)} \cdot i d f(t) \\
\qquad i d f(t)=\log \frac{N-d f(t)+0.5}{d f(t)+0.5}
\end{array}
$$



* $$boost$$：相应域的权值
* $$l_c$$：field的长度
* $$avl_c$$：field的平均长度
* $$b_c$$：调节因子

BM25F的最终值，就是各个field的分值的加权求和。