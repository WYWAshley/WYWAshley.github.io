---
layout: post
title: 多元高斯分布
categories: [mathematics, probability]
description: 介绍多元高斯分布的由来与其背后的几何原理
keywords: gaussian distribution
---

介绍多元高斯分布的由来与其背后的几何原理

1. 阐述多元标准高斯分布;
2. 由多元标准高斯分布导出多元高斯分布;
3. 阐述多元高斯分布的几何意义;
4. 总结.

<br>

## 多元标准高斯分布

熟悉一元高斯分布的同学都知道, 若随机变量 ![[公式]](https://www.zhihu.com/equation?tex=X++%5Csim+%5Cmathcal%7BN%7D%28%5Cmu%2C+%5Csigma%5E2%29) , 则有如下的概率密度函数

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+p%28x%29+%26+%3D+%5Cfrac%7B1%7D%7B%5Csigma+%5Csqrt%7B2+%5Cpi%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28%5Cfrac%7Bx+-+%5Cmu%7D%7B%5Csigma%7D%29%5E2%7D+%5Ctag%7B1%7D+%5C%5C+1+%26+%3D+%5Cint_%7B-+%5Cinfty%7D%5E%7B%2B+%5Cinfty%7D%7Bp%28x%29dx%7D+%5Ctag%7B2%7D+%5Cend%7Balign%7D)

而如果我们对随机变量 ![[公式]](https://www.zhihu.com/equation?tex=X) 进行标准化, 用 ![[公式]](https://www.zhihu.com/equation?tex=Z+%3D+%5Cfrac%7BX+-+%5Cmu%7D%7B%5Csigma%7D) 对(1)进行换元, 继而有

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cbecause+x%28z%29+%26+%3D+z+%5Ccdot+%5Csigma+%2B+%5Cmu+%5C%5C+%5Ctherefore+p%28x%28z%29%29+%26+%3D+%5Cfrac%7B1%7D%7B%5Csigma+%5Csqrt%7B2+%5Cpi%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28z%29%5E2%7D+%5C%5C+1+%26+%3D+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D%7Bp%28x%28z%29%29dx%7D+%5C%5C+%26+%3D++%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D%7B+%5Cfrac%7B1%7D%7B%5Csigma+%5Csqrt%7B2+%5Cpi%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28z%29%5E2%7Ddx%7D+%5C%5C+++%26+%3D++%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D%7B+%5Cfrac%7B1%7D%7B%5Csqrt%7B2+%5Cpi%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28z%29%5E2%7D+dz%7D+%5Ctag%7B3%7D+%5Cend%7Balign%7D+)

此时我们说随机变量 ![[公式]](https://www.zhihu.com/equation?tex=Z+%5Csim+%5Cmathcal%7BN%7D%280%2C+1%29) 服从一元标准高斯分布, 其均值 ![[公式]](https://www.zhihu.com/equation?tex=%5Cmu+%3D+0) , 方差 ![[公式]](https://www.zhihu.com/equation?tex=%5Csigma%5E2+%3D+1) , 其概率密度函数为

![[公式]](https://www.zhihu.com/equation?tex=p%28z%29+%3D+%5Cfrac%7B1%7D%7B%5Csqrt%7B2+%5Cpi%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28z%29%5E2%7D+%5Ctag4)

需要注意的是, 为了保证概率密度函数在 ![[公式]](https://www.zhihu.com/equation?tex=R) 上的积分为1, 换元时需要求 ![[公式]](https://www.zhihu.com/equation?tex=dx+%3D+%5Csigma+%5Ccdot+dz) , 从而得到(3).

**随机变量** ![[公式]](https://www.zhihu.com/equation?tex=X) **标准化的过程, 实际上的消除量纲影响和分布差异的过程. 通过将随机变量的值减去其均值再除以标准差, 使得随机变量与其均值的差距可以用若干个标准差来衡量, 从而实现了不同随机变量与其对应均值的差距, 可以以一种相对的距离来进行比较.**

一元标准高斯分布与我们讨论多元标准高斯分布有什么关系呢? 事实上, 多元标准高斯分布的概率密度函数正是从(4)导出的. 假设我们有随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+%5BZ_1%2C+%5Ccdots%2C+Z_n%5D%5E%5Ctop) , 其中 ![[公式]](https://www.zhihu.com/equation?tex=Z_i+%5Csim+%5Cmathcal%7BN%7D%280%2C+1%29+%28i+%3D+1%2C+%5Ccdots%2C+n%29) 且 ![[公式]](https://www.zhihu.com/equation?tex=Z_i%2C+Z_j%28i%2C+j+%3D+1%2C+%5Ccdots%2C+n+%5Cwedge+i+%5Cneq+j%29) 彼此独立, 即随机向量中的每个随机变量 ![[公式]](https://www.zhihu.com/equation?tex=Z_%7Bi%7D) 都服从标准高斯分布且两两彼此独立. 则由(4)与独立随机变量概率密度函数之间的关系, 我们可得随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+%5BZ_1%2C+%5Ccdots%2C+Z_n%5D%5E%5Ctop) 的联合概率密度函数为

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+p%28z_1%2C+%5Ccdots%2C+z_n%29+%26+%3D+%5Cprod_%7Bi%3D1%7D%5E%7Bn%7D+%5Cfrac%7B1%7D%7B%5Csqrt%7B2+%5Cpi%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28z_i%29%5E2%7D+%5C%5C++%26+%3D+%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28Z%5E%5Ctop+Z%29%7D++%5C%5C+1+%26+%3D+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D+%5Ccdots+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D+p%28z_1%2C+%5Ccdots%2C+z_n%29+%5C+dz_1+%5Ccdots+dz_n+%5Ctag5+%5Cend%7Balign%7D)

我们称随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B0%7D%2C+%5Cmathbf%7BI%7D%29) , 即随机向量服从均值为零向量, 协方差矩阵为单位矩阵的高斯分布. 在这里, 随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D) 的协方差矩阵是 ![[公式]](https://www.zhihu.com/equation?tex=Conv%28Z_i%2C+Z_j%29%2C+i%2C+j+%3D+1%2C+%5Ccdots%2C+n) 组成的矩阵, 即

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5BConv%28Z_i%2C+Z_j%29%5D_%7Bn+%5Ctimes+n%7D+%26+%3D+%5Cmathbf%7BE%7D%5B%28Z+-+%5Cvec%7B%5Cmu%7D%29%28Z+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop%5D+%5C%5C+%26+%3D+%5Cmathbf%7BI%7D+%5Ctag6+%5Cend%7Balign%7D)

由于随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B0%7D%2C+%5Cmathbf%7BI%7D%29) , 所以其协方差矩阵的对角线元素为1, 其余元素为0. 如果我们取常数 ![[公式]](https://www.zhihu.com/equation?tex=c+%3D+p%28z_1%2C+%5Ccdots%2C+z_n%29) , 则可得函数 ![[公式]](https://www.zhihu.com/equation?tex=p%28z_1%2C+%5Ccdots%2C+z_n%29) 的等高线为 ![[公式]](https://www.zhihu.com/equation?tex=c%27+%3D+Z%5E%5Ctop+Z) , 当随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D) 为二维向量时, 我们有

![[公式]](https://www.zhihu.com/equation?tex=c%27+%3D+Z%5E%5Ctop+%5Ccdot+Z+%3D+%28z_1+-+0%29%5E2+%2B+%28z_2+-+0%29%5E2+%5Ctag7)

由(7)我们可知, 其等高线为以(0, 0)为圆心的同心圆.

<img src="/images/posts/Multivariate Gaussian Distribution/1.png"/>

<br>

## 多元高斯分布

由上一节我们知道, 当随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B0%7D%2C+%5Cmathbf%7BI%7D%29) 时, 其每个随机变量 ![[公式]](https://www.zhihu.com/equation?tex=Z_i+%5Csim+%5Cmathcal%7BN%7D%280%2C+1%29+%28i+%3D+1%2C+%5Ccdots%2C+n%29) 彼此独立, 我们可通过(4)与独立随机变量概率密度函数之间的关系得出其联合概率密度函数(5). 那对于普通的随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B%5Cmu%7D%2C+%5CSigma%29) , 即其每个随机变量 ![[公式]](https://www.zhihu.com/equation?tex=X_i+%5Csim+%5Cmathcal%7BN%7D%28%5Cmu_i%2C+%5Csigma_i%5E2%29+%28i+%3D+1%2C+%5Ccdots%2C+n%29) 且 ![[公式]](https://www.zhihu.com/equation?tex=X_i%2C+X_j%28i%2C+j+%3D+1%2C+%5Ccdots%2C+n%29) 彼此不独立的情况下, 我们该如何求随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 的联合概率密度函数呢? 一个很自然的想法是, **如果我们能通过线性变换, 使得随机向量** ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+%3D+%5BX_1%2C+%5Ccdots%2C+X_n%5D%5E%5Ctop) **中的每个随机变量彼此独立, 则我们也可以通过独立随机变量概率密度函数之间的关系求出其联合概率密度函数**. 事实上, 我们有如下定理可完成这个工作 ![[公式]](https://www.zhihu.com/equation?tex=%5E%7B%5B2%5D%7D)

> 定理1: 若存在随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B%5Cmu%7D%2C+%5CSigma%29) , 其中 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7B%5Cmu%7D+%5Cin+R%5En) 为均值向量, ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma+%5Cin+S%5E%7Bn+%5Ctimes+n%7D_%7B%2B%2B%7D) 半正定实对称矩阵为 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 的协方差矩阵, 则存在满秩矩阵 ![[公式]](https://www.zhihu.com/equation?tex=B+%5Cin+R%5E%7Bn+%5Ctimes+n%7D) , 使得 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) , 而 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B0%7D%2C+%5Cmathbf%7BI%7D%29) .

有了定理1, 我们就可以对随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 做相应的线性变换, 使其随机变量在线性变换后彼此独立, 从而求出其联合概率密度函数, 具体地

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%26+%5Cbecause+%26+%5Cvec%7BZ%7D+%26+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%2C+%5Cvec%7BZ%7D++%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B0%7D%2C+I%29+%5Ctag8+%5C%5C++%26+%5Ctherefore+%26+p%28z_1%2C+%5Ccdots%2C+z_n%29+%26+%3D+%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%28Z%5E%5Ctop+Z%29%7D+%5C%5C+%26+%26+p%28z_1%28x_1%2C+%5Ccdots%2C+x_n%29%2C+%5Ccdots%29+%26+%3D+%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%5B%28B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%29%5E%5Ctop+%28B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%29%5D%7D+%5C%5C+%26+%26+%26+%3D+%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%5B%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%28BB%5E%5Ctop%29%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%7D+%5C%5C+%26+%5Ctherefore+%26+1+%3D+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D+%26+%5Ccdots+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D+p%28z_1%28x_1%2C+%5Ccdots%2C+x_n%29%2C+%5Ccdots%29+%5C+dz_1+%5Ccdots+dz_n+%5C%5C+%26++%26+%3D+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D+%26+%5Ccdots++%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D++%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%5B%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%28BB%5E%5Ctop%29%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%7D++%5C+dz_1+%5Ccdots+dz_n+%5Ctag9+%5Cend%7Balign%7D)

由多元函数换元变换公式, 我们还需要求出雅可比行列式 ![[公式]](https://www.zhihu.com/equation?tex=J%28%5Cvec%7BZ%7D+%5Cto+%5Cvec%7BX%7D%29) , 由(8)可得

![[公式]](https://www.zhihu.com/equation?tex=J%28%5Cvec%7BZ%7D+%5Cto+%5Cvec%7BX%7D%29+%3D+%5Cleft%7C+B%5E%7B-1%7D+%5Cright%7C+%3D+%5Cleft%7C+B+%5Cright%7C%5E%7B-1%7D+%3D++%5Cleft%7C+B+%5Cright%7C%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D++%5Ccdot+%5Cleft%7C+B%5E%5Ctop+%5Cright%7C%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D+%3D+%5Cleft%7C+B+B%5E%5Ctop+%5Cright%7C%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D+%5Ctag%7B10%7D)

由(9)(10), 我们可进一步得

![[公式]](https://www.zhihu.com/equation?tex=1+%3D+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D+%5Ccdots+%5Cint_%7B-%5Cinfty%7D%5E%7B%2B%5Cinfty%7D++%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D+%5Cleft%7C+B+B%5E%5Ctop+%5Cright%7C%5E%7B%5Cfrac%7B1%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%5B%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%28BB%5E%5Ctop%29%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%7D++%5C+dx_1+%5Ccdots+dx_n+%5Ctag%7B11%7D)

我们得到随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B%5Cmu%7D%2C+%5CSigma%29) 的联合概率密度函数为

![[公式]](https://www.zhihu.com/equation?tex=p%28x_1%2C+%5Ccdots%2C+x_n%29+%3D+%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D+%5Cleft%7C+B+B%5E%5Ctop+%5Cright%7C%5E%7B%5Cfrac%7B1%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%5B%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%28BB%5E%5Ctop%29%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%7D+%5Ctag%7B12%7D)

在(12)中, 随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 的协方差矩阵还未得到体现, 我们可通过线性变换(8)做进一步处理

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5CSigma+%26+%3D+%5Cmathbf%7BE%7D%5B%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop%5D+%5C%5C+%26+%3D+%5Cmathbf%7BE%7D%5B%28B%5Cvec%7BZ%7D+-+%5Cvec%7B0%7D%29%28B%5Cvec%7BZ%7D+-+%5Cvec%7B0%7D%29%5E%5Ctop%5D+%5C%5C+%26+%3D+%5Cmathbf%7BConv%7D%28B%5Cvec%7BZ%7D%2C+B%5Cvec%7BZ%7D%29+%5C%5C+%26+%3D+B++%5Cmathbf%7BConv%7D%28%5Cvec%7BZ%7D%2C+%5Cvec%7BZ%7D%29+B%5E%5Ctop+%5C%5C+%26+%3D+BB%5E%5Ctop+%5Ctag%7B13%7D+%5Cend%7Balign%7D)

我们发现, (12)中 ![[公式]](https://www.zhihu.com/equation?tex=BB%5E%5Ctop) 就是线性变换前的随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B%5Cmu%7D%2C+%5CSigma%29) 的协方差矩阵 ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma) , 所以由(12)(13), 我们可以得到联合概率密度函数的最终形式

![[公式]](https://www.zhihu.com/equation?tex=p%28x_1%2C+%5Ccdots%2C+x_n%29+%3D+%5Cfrac%7B1%7D%7B%282+%5Cpi%29%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D+%5Cleft%7C+%5CSigma+%5Cright%7C%5E%7B%5Cfrac%7B1%7D%7B2%7D%7D%7D+%5Ccdot+e%5E%7B-%5Cfrac%7B1%7D%7B2%7D+%5Ccdot+%5B%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%5CSigma%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%7D+%5Ctag%7B14%7D)

**原本由定理1, 我们还需要求线性变换矩阵** ![[公式]](https://www.zhihu.com/equation?tex=B) **, 才能确定随机向量** ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) **的联合概率密度函数的表达式, 现在由(13)我们即可得最终形式(14), 随机向量** ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) **的联合概率密度函数由其均值向量** ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7B%5Cmu%7D) **和其协方差矩阵** ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma) **唯一确定, 但我们需要明白的是, 这是通过定理1的线性变换** ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) **得到的, 即此线性变换隐含其中.**

如果我们取常数 ![[公式]](https://www.zhihu.com/equation?tex=c+%3D+p%28x_1%2C+%5Ccdots%2C+x_n%29) , 则可得函数 ![[公式]](https://www.zhihu.com/equation?tex=p%28x_1%2C+%5Ccdots%2C+x_n%29) 的等高线为![[公式]](https://www.zhihu.com/equation?tex=c%27+%3D+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%5CSigma%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) , 当随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 为二维向量时, 我们对协方差矩阵 ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma) 进行分解, 因为其为实对称矩阵, 可正交对角化

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5CSigma+%26+%3D+Q+%5CLambda+Q%5E%5Ctop+%5C%5C+c%27+%26+%3D+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%28Q+%5CLambda+Q%5E%5Ctop%29%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29+%5C%5C+%26+%3D+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+Q+%5CLambda%5E%7B-1%7D+Q%5E%5Ctop%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29+%5C%5C++%26+%3D+%5BQ%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%5E%5Ctop+%5CLambda%5E%7B-1%7D+%5BQ%5E%5Ctop%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D+%5Ctag%7B15%7D++%5Cend%7Balign%7D)

由于矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 是酉矩阵, 所以 ![[公式]](https://www.zhihu.com/equation?tex=Q%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29+%3D+Q%5E%5Ctop+%5Cvec%7BX%7D+-+Q%5E%5Ctop+%5Cvec%7Bu%7D) 可以理解为将随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) , 均值向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7B%5Cmu%7D) 在矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 的列向量所组成的单位正交基上进行投影并在该单位正交基上进行相减. 我们不妨记投影后的向量分别为 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D_Q+%3D+Q%5E%5Ctop+%5Cvec%7BX%7D%2C+%5Cvec%7Bu%7D_Q+%3D+Q%5E%5Ctop+%5Cvec%7B%5Cmu%7D) , 同时记矩阵 ![[公式]](https://www.zhihu.com/equation?tex=%5CLambda+%3D+%5Cbegin%7Bbmatrix%7D+%5Clambda_1+%26+0+%5C%5C+0+%26+%5Clambda_2%5Cend%7Bbmatrix%7D%2C+%5Clambda_1+%5Cgeq+%5Clambda_2) , 则(15)的二次型可表示为

![[公式]](https://www.zhihu.com/equation?tex=c%27+%3D+%28%5Cfrac%7BX_%7BQ_1%7D+-+%5Cmu_%7BQ_1%7D%7D%7B%5Csqrt%7B%5Clambda_1%7D%7D%29%5E2+%2B+%28%5Cfrac%7BX_%7BQ_2%7D+-+%5Cmu_%7BQ_2%7D%7D%7B%5Csqrt%7B%5Clambda_2%7D%7D%29%5E2+%5Ctag%7B16%7D)

由(16)我们可知, 此时函数 ![[公式]](https://www.zhihu.com/equation?tex=p%28x_1%2C+%5Ccdots%2C+x_n%29) 的等高线是在矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 的列向量所组成的单位正交基上的一个椭圆, 椭圆的中心是 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7Bu%7D_%7BQ%7D+%3D+%5B%5Cmu_%7BQ_1%7D%2C+%5Cmu_%7BQ_2%7D%5D%5E%5Ctop) , 长半轴为 ![[公式]](https://www.zhihu.com/equation?tex=%5Csqrt%7B%5Clambda_1%7D) , 短半轴为 ![[公式]](https://www.zhihu.com/equation?tex=%5Csqrt%7B%5Clambda_%7B2%7D%7D) .

如果协方差矩阵 ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma) 不是对角矩阵, 则正交对角化得到的酉矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 不是标准正交基, 其代表一个旋转, 此时的椭圆应该是一个倾斜的椭圆, 随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 中的随机变量不是彼此独立的;

<img src="/images/posts/Multivariate Gaussian Distribution/2.png"/>  

如果协方差矩阵 ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma) 是对角矩阵, 则正交对角化得到的酉矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 就是标准正交基, 则前述的投影是在标准正交基上完成的, 此时的椭圆应该是一个水平的椭圆, 随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 中的随机变量就是彼此独立的.

<img src="/images/posts/Multivariate Gaussian Distribution/3.png"/>

<br>

## 多元高斯分布的几何意义

现在我们知道, 随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B%5Cmu%7D%2C+%5CSigma%29) 的联合概率密度函数是通过线性变换 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) 的帮助, 将随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 的各个随机变量去相关性, 然后利用独立随机变量概率密度函数之间的关系得出的, 亦既是定理1所表述的内容. 那具体地, 线性变化 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) 是怎么去相关性使随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 的各个随机变量彼此独立的呢? 我们不妨在二维平面上, 再次由定理1和(15)出发来看看这个去相关性的过程.

由定理1我们有

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%26+%5Cbecause+%26+%5Cvec%7BZ%7D+%26+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%2C+%5Cvec%7BZ%7D++%5Csim+%5Cmathcal%7BN%7D%28%5Cvec%7B0%7D%2C+I%29+%5C%5C+%26%5Ctherefore+%26+%5Cvec%7BZ%7D%5E%5Ctop+%5Cvec%7BZ%7D+%26+%3D+%28B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%29%5E%5Ctop+%28B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%29+%5C%5C+%26+%26+%26+%3D+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%28BB%5E%5Ctop%29%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29+%5C%5C+%26+%26+%26+%3D+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5E%5Ctop+%5CSigma%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29+%5Ctag%7B17%7D+%5Cend%7Balign%7D)

再由(15)(17)可得

![[公式]](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cvec%7BZ%7D%5E%5Ctop+%5Cvec%7BZ%7D+%26+%3D+%5BQ%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%5E%5Ctop+%5CLambda%5E%7B-1%7D+%5BQ%5E%5Ctop%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D+%5C%5C+%26+%3D+%5BQ%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%5E%5Ctop+%28%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D%29%5E%5Ctop+%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D+%5BQ%5E%5Ctop%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D+%5C%5C+%26+%3D+%5B%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7DQ%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%5E%5Ctop+%5B%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D+Q%5E%5Ctop%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D+%5C%5C++%26+%3D+%5B%28Q+%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D%29%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%5E%5Ctop+%5B%28Q+%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D%29%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D+%5C%5C+%26+%3D+%5B%28Q+%5Ccdot+%5Cbegin%7Bbmatrix%7D+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_1%7D%7D+%26+0+%5C%5C+0+%26+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_2%7D%7D+%5Cend%7Bbmatrix%7D%29%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D%5E%5Ctop+%5C%5C+%26+%5Ccdot+%5C++%5C+%5B%28Q+%5Ccdot+%5Cbegin%7Bbmatrix%7D+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_1%7D%7D+%26+0+%5C%5C+0+%26+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_2%7D%7D+%5Cend%7Bbmatrix%7D%29%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%5D++%5Ctag%7B18%7D+%5Cend%7Balign%7D)

由(18)我们已经可以非常明显地看出线性变换 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) 的具体操作了

![[公式]](https://www.zhihu.com/equation?tex=%5Cunderbrace%7B%28%5Cunderbrace%7BQ+%5Ccdot+%5Cunderbrace%7B%5Cbegin%7Bbmatrix%7D+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_1%7D%7D+%26+0+%5C%5C+0+%26+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_2%7D%7D+%5Cend%7Bbmatrix%7D%7D_%7B%E5%AF%B9%E6%A0%87%E5%87%86%E6%AD%A3%E4%BA%A4%E5%9F%BA%E6%8B%89%E4%BC%B8%7D%7D_%7B%E5%AF%B9%E6%8B%89%E4%BC%B8%E5%90%8E%E7%9A%84%E6%AD%A3%E4%BA%A4%E5%9F%BA%E6%97%8B%E8%BD%AC%7D+%29%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29%7D_%7B%E5%B0%86%E9%9A%8F%E6%9C%BA%E5%90%91%E9%87%8F%5Cvec%7BX%7D%E5%8E%BB%E5%9D%87%E5%80%BC%E5%90%8E%E5%9C%A8%E6%96%B0%E6%AD%A3%E4%BA%A4%E5%9F%BA%E4%B8%8A%E6%8A%95%E5%BD%B1%7D+%5Ctag%7B19%7D)

我们先对标准正交基进行拉伸, 横轴和纵轴分别拉伸 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_1%7D%7D%2C+%5Cfrac%7B1%7D%7B%5Csqrt%7B%5Clambda_2%7D%7D) 倍, 再使用酉矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 对拉伸后的正交基进行旋转, 最后将去均值的随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D) 在新的正交基上进行投影, 从而使完成线性变换 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BZ%7D+%3D+B%5E%7B-1%7D%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) 后的随机变量在新的正交基上彼此独立. 值得注意的是, 如果随机向量 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) 本来就是独立随机变量组成的, 此时其协方差矩阵是一个对角矩阵, 则酉矩阵 ![[公式]](https://www.zhihu.com/equation?tex=Q) 是一个单位矩阵 ![[公式]](https://www.zhihu.com/equation?tex=%5Cmathbf%7BI%7D) , 此线性变换中只有拉伸而没有旋转.

<img src="/images/posts/Multivariate Gaussian Distribution/4.png"/>

**而如果我们只保留** ![[公式]](https://www.zhihu.com/equation?tex=%28Q+%5CLambda%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7D%29%5E%5Ctop+%28%5Cvec%7BX%7D+-+%5Cvec%7B%5Cmu%7D%29) **这个投影后坐标轴长度较长的对应的坐标, 我们就可以达到将随机向量** ![[公式]](https://www.zhihu.com/equation?tex=%5Cvec%7BX%7D) **进行降维的效果, 而这, 就是所谓的PCA(principal component analysis, 主成分分析).**

<br>

## 总结

本文从多元标准高斯分布出发, 阐述了如何通过线性变换, 将任意的服从多元高斯分布的随机向量去相关性, 并求出其联合概率密度函数的过程, 最后给出了线性变换的具体过程阐述. 多元高斯分布是许多其他理论工具的基础, 掌握它是进行其他相关理论研究的关键.