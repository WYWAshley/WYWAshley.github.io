---
layout: post
title: 逻辑回归和逻辑斯蒂函数
categories: [机器学习, 算法]
description: understand why logistic regression
keywords: logistic regression
---

逻辑回归与线性回归的区别，以及 logistic 函数的意义

## 一、为什么要使用逻辑回归？ 　　

​	都说线性回归用来做回归预测，逻辑回归用于做二分类，一个是解决回归问题，一个用于解决分类问题。但很多人问起逻辑回归和线性回归的区别，很多人会说：逻辑回归就是对线性回归做了一个压缩，**将 $y$ 的阈值从 $y∈(+∞,−∞)$ 压缩到 $(0,1)$**。那么问题来了，问什么仅仅做一个简单的压缩，就将回归问题变成了分类问题？里面蕴含着本质？ 

​	首先要从数据说起，线性回归的样本的输出，都是连续值，y∈(+∞,−∞)而，逻辑回归中y∈{0,1}，只能取0和1。对于拟合函数也有本质上的差别： 　　

​	线性回归：$f(x)=θ^TX=θ_1x_1+θ_2x_2+⋯+θ_nx_n$

​	逻辑回归：$f(x)=p(y=1∣x;θ)=g(θTX)$，其中 $g(z)=\frac{1}{1+e^-z}$

​	可以看出，线性回归的拟合函数，的确是对f(x)的输出变量y的拟合，而逻辑回归的拟合函数是对为1类的样本的概率的拟合。

 <br>

## 二、那么，为什么逻辑回归用 logistic 函数进行拟合呢？ 　　

​	首先，logstic 函数的本质说起。若要直接通过回归的方法去预测二分类问题， y 到底是0类还是1类，最好的函数是单位阶跃函数。**然而单位阶跃函数不连续（GLM 的必要条件），而 logsitic 函数恰好接近于单位阶跃函数，且单调可微。**于是希望通过该复合函数去拟合分类问题：$y$，于是有：
$$
y=\frac{1}{1+e^{-\theta^{T}X}} \\
ln\frac{y}{1-y}=\theta^TX
$$
发现如果我们假设 $y=p(y为1类∣x;θ) $作为我们的拟合函数，等号左边的表达式的数学意义就是1类和0类的对数几率（log odds）。

这个表达式的意思就是：**用线性模型的预测结果去逼近1类和0类的几率比。**于是，θTX=0就相当于是1类和0类的决策边界： 　　

当 $θTX>0$，则有 $y>0.5$；若 $θTX→+∞$,则y→1 ，即 $y$ 为1类; 　　

当 $θTX<0$，则有 $y<0.5$ ;若 $θTX→−∞$，则y→0，即 $y$ 为0类。 　

这个时候就能看出区别来了，在线性回归中θTX为预测值的拟合函数；而在逻辑回归中 $θTX=0$ 为决策边界。