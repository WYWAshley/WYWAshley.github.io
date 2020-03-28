---
layout: post
title: Activation Function
categories: [machine learning, algorithm]
description: 
keywords: keyword1, keyword2
---

为什么要用到激活函数？激活函数的评论法则？常用的激活函数？

## 一、使用激活函数的原因

* 类似于**神经突触**：如果我们不运用激活函数的话，则输出信号将仅仅是一个**简单的线性函数**，它功率有限，并且大多数情况下执行得并不好。我们希望我们的神经网络不仅仅可以学习和计算线性函数，而且还要比这复杂得多，例如图像、视频、音频、语音等。这就是为什么我们要使用人工神经网络技术，诸如深度学习（Deep learning），来理解一些复杂的事情，一些相互之间具有很多隐藏层的非线性问题，而这也可以帮助我们了解复杂的数据。
* 为保证非线性，激活函数必须为非线性函数，但仅仅具有非线性是不够的。神经网络在本质上是一个复合函数，这会让我们思考一个问题：这个函数的建模能力有多强？即它能模拟什么样的目标函数？已经证明，只要激活函数选择得当，神经元个数足够多，使用3层即包含一个隐含层的神经网络就可以实现对任何一个从输入向量到输出向量的连续映射函数的逼近，这个结论称为**万能逼近（universal approximation）定理**。万能定理对激活函数的要求是必须**非常数、有界、单调递增并且连续可导**。

<br>

## 二、激活函数的评论法则

​	好的激活函数一般拥有以下三个特点：

* **Unboundedness(x>0无饱和区域)**：$\lim_{x\to+\infty}f′(x)=0$ （这就是右饱和）传统的激活函数如 sigmoid 和 tanh 函数都有饱和区域，所以必须依赖较好的初始化让输入数据处于非饱和区域，否则饱和区域产生的梯度过小会影响收敛速度，而 Relu 系列都是 x>0 无饱和区域。
* **NegativeOutputs(x<0产生非0值)**：Relu 在 x<0 的值全都是 0，而 PReLU，RReLU，LeakyReLU 最大的共同改进点就是在 x<0 产生非0值，少量的 NegativeOutputs 能减少神经元训练过程中出现"die"的概率，提升模型的鲁棒性。
* **Smoothness(平滑性)**：Relu，PReLU，RReLU 都是阶跃函数，阶跃函数在特征响应图上最明显的现象是断层，而平滑的激活函数更利于梯度信息的回传。对于目标函数，通常存在梯度变化很大的一个“悬崖”，在此处求梯度，很容易导致求解不稳定的梯度爆炸现象。

<img src="/images/posts/Activation Function/3.png"/>

### 梯度消失和梯度爆炸

假设每层只有一个神经元（这里的 b 不是 bias，是代表神经元），激活函数选sigmoid函数，第 j 个神经元输入输出分别为 $z_j=w_ja_{j−1}+b_j \ \ \ \ (a是上一层的结果) \ \ \ \ \ \  a_j=σ(z_j)$：

<img src="/images/posts/Activation Function/1.png"/>

则，对 $b1$ (这里的 b 是 bias)的一个小变化引起 $C$ 的变化为：$\frac{∂C}{∂b_1}≈\frac{ΔC}{Δb_1}$

$b1$ 的变化引起 $a1$ 的变化为： $a_1=σ(z_1)=σ(w_1a_0+b_1)$      $Δa_1≈\frac{∂w_1a_0+b_1}{∂b_1}Δb1=σ′(z_1)Δb_1$

$a1$ 的变化引起 $z2$ 的变化为：$z_2=w_2a_1+b_2$   $Δz_2≈\frac{∂z_2}{∂a_1}Δa_1=w_2Δa_1$

把以上 $a1$的变化代入 $z2$ 的变化：$Δa_1≈σ′(z_1)w_2Δb_1$

依次类推至 $z3 z4$ 的变化一直到输出层，得到：
$$
ΔC≈σ′(z_1)w_2σ′(z_2)w_3σ′(z_3)w_4σ′(z_4)\frac{∂C}{∂a_4}Δb_1
$$
等式两边同时除以 $b1$ 的变化，得到：
$$
\frac{∂C}{∂b_1}≈σ′(z_1)w_2σ′(z_2)w_3σ′(z_3)w_4σ′(z_4)\frac{∂C}{∂a_4}
$$

由 sigmoid 函数的导数 $σ′$ 的图像可以看出，函数的最大值 σ′(0)=0.25;

<img src="/images/posts/Activation Function/2.png"/>

这样可以看到，如果我们使用标准化初始w，那么各个层次的相乘都是0-1之间的小数，而激活函数f的导数也是0-1之间的数，其连乘后，结果会变的很小，导致**梯度消失**。若我们初始化的w是很大的数，w大到乘以激活函数的导数都大于1，那么连乘后，可能会导致求导的结果很大，形成**梯度爆炸**。

按平时随机从高斯分布N(0,1)中随机产生权重的方法，大部分 $|w|<1 ， |w_jσ′(z_j)|<\frac{1}{4} $；

则由 $\frac{∂C}{∂b1}≈σ′(z_1)w_2σ′(z_2)w_3σ′(z_3)w_4σ′(z_4)\frac{∂C}{∂a_4}$

以及 $\frac{∂C}{∂b_3}≈σ′(z_3)w_4σ′(z_4)\frac{∂C}{∂a_4}$

可以看出层数越多连续乘积越小，所以梯度消失更容易出现。若要出现梯度爆炸，其w既要大还要保证激活函数的导数不要太小。

<br>

## 三、常用的激活函数

#### sigmoid

sigmoid函数的表达式为：
$$
σ(x)=\frac{1}{1+e^{-x}}
$$
其函数图像如下图所示：

<img src="/images/posts/Activation Function/4.png"/>

sigmoid的值域为(0,1)。其导数为：
$$
σ′(x)=\sigma(x)(1 - \sigma(x))
$$


* **求导容易**，利用其本身函数值即可。

* 函数具有软饱和性，容易产生**梯度消失**问题。
* **输出不以 0 为中心**，如果数据输入神经元是正的，那么会导致计算的梯度也始终为正。
* $e^{-x}$ 计算耗时。

<br>

#### tanh

tanh 函数的表达式为：
$$
tanh(x)=\frac{e^x-e^{-x}}{e^x+e^{-x}}
$$
其函数图像如下图所示：

<img src="/images/posts/Activation Function/5.png"/>

tanh的值域为 (-1, 1)。其导数为：
$$
tanh'(x)=1-(tanh(x))^2
$$

* 输出以0为中心。
* 比sigmoid函数训练时收敛更快。
* 仍然是饱和函数，没有解决梯度消失问题。

<br>

#### ReLU

ReLU函数的表达式为：
$$
ReLU(x)=max(0,x)
$$
其函数图像如下图所示：

<img src="/images/posts/Activation Function/6.png"/>

其导数为：
$$
ReLU'(x)=sign(x)=\begin{cases} 0 & x<0 \\ 1 & x\geq 0 \end{cases}
$$
需要注意的是，ReLU 在原点处不可导。

* 解决了部分梯度消失问题
* 训练收敛速度相比tanh更快
* 当 $x<0$ 时，出现梯度消失问题。此时相当于神经元死亡。
  假设神经网络某层的计算公式为 $y=ReLU(WTX+b)$ 如果学习率设置的比较大，且此时的实际值为负例，训练结果为使模型往实际值拟合，那么权重W就会突然变得很小，使 $WTX+b$ 为负，再经过 ReLU ，那么输出就为0。但反向传播到此处就会碰到问题，因为此时梯度为0，从而无法更新参数值，也就相当于神经元死亡了。
  对于这种问题的解决办法一般是设置一个比较小的学习率，或者使用L2正则化以及使用Momentum、RMS、Adam等其他梯度下降方式。

<br>

#### Leaky ReLU

是对 ReLU 的改进，表达式为
$$
f(x)=\begin{cases} x & x \geq 0 \\ \alpha x & x<0 \end{cases}
$$
$\alpha$ 是一个很小的正数，比如 0.01，那么在 $(-\infty,0]$ 范围的导数不为 0，解决了 ReLU神经元死掉的问题。

<br>

#### PReLU

PReLU(Parametric Rectified Linear Unit), 顾名思义：带参数的 ReLU。二者的定义和区别如下图： 

<img src="/images/posts/Activation Function/7.png"/>

如果 $a_i=0$，那么 PReLU 退化为 ReLU；如果 $a_i$ 是一个很小的固定值(如 $a_i=0.01$)，则 PReLU 退化为 Leaky ReLU(LReLU)。 有实验证明，与 ReLU 相比，LReLU 对最终的结果几乎没什么影响。

PReLU 只增加了极少量的参数，也就意味着网络的计算量以及过拟合的危险性都只增加了一点点。特别的，当不同 channels 使用相同的 $a_i$ 时，参数就更少了。

BP更新 $a_i$ 时，采用的是带动量的更新方式，如下图：

<img src="/images/posts/Activation Function/9.png"/>

上式的两个系数分别是动量和学习率。 

普通的梯度下降算法在寻找最优解的过程中会酱紫：

<img src="/images/posts/Activation Function/8.png"/>

可以看到是存在不断抖动的

使用了带动量的梯度下降，由于梯度的计算使用了**指数加权平均方法，使得本次梯度的计算和之前是有关联的**，这样就能抵消比如梯度在上下摆动的这种状况，而真正的下降方向（朝右边走）却能很好保持，这样使得收敛优化变得更快

需要特别注意的是：**更新 $a_i$ 时不施加权重衰减(L2正则化)**，因为这会把 $a_i$ 很大程度上 push 到 0。事实上，即使不加正则化，试验中 $a_i$ 也很少有超过1的。

<br>

#### Randomized Leaky ReLU

Randomized Leaky ReLU 中的 $\alpha$ 在训练阶段是从高斯分布 $U(l,u)$ 中随机取出来的，然后在测试过程中进行纠正。
$$
y_{ji}=\begin{cases} x_{ji}, & x_{ji}\geq 0\\ \alpha_{ji}x_{ji}, & x_{ji}<0 \end{cases}
$$
<img src="/images/posts/Activation Function/10.png"/>

<br>

#### Maxout

$$
f_i(x)=max_{i\in[1,k]}{z_{ij}}
$$

假设 w 是二维的，那么 $f(x)=max(w_1^TX+b_1, w_2^TX+b_2)$

ReLu 和 Leaky ReLU 都是它的变形（比如 $w_1, b_1 = 0 $ 的时候，就是 ReLU ）。
Maxout 的拟合能力是非常强的，它可以拟合任意的凸函数，最直观的解释就是任意的凸函数都可以由分段线性函数以任意精度拟合，前提是“隐含层”的神经元个数可以任意多。

<img src="/images/posts/Activation Function/11.png"/>

- 计算简单。
- 解决了梯度消失问题，且不会产生死亡神经元。
- 训练收敛速度快。

- 每个神经元的参数加倍，导致参数数量激增。

<br>

## 四、如何选择

　　1、对于RNN，可以通过梯度截断，避免梯度爆炸

　　2、可以通过添加正则项，避免梯度爆炸

　　3、使用LSTM等自循环和门控制机制，避免梯度消失，参考：https://www.cnblogs.com/pinking/p/9362966.html

　　4、优化激活函数，譬如将sigmold改为relu，避免梯度消失