---
layout: post
title: Numpy库的各种使用方法
categories: [python]
description: 列举一些numpy库的常用方法
keywords: numpy
---

列举一些numpy库的常用方法：创建数组，改变数组形状，复制数组，数组堆叠，数组切分，数组重复，随机数等等。

<br>

## 数组的基本信息

```python
import numpy as np

ar = np.array([[1,2,3,4,5,6,7], [1,2,3,4,5,6,7]])
print(ar)          # 输出数组，注意数组的格式：中括号，元素之间没有逗号（和列表区分）
print(ar.ndim)     # 输出数组维度的个数（轴数），或者说“秩”，维度的数量也称rank
print(ar.shape)    # 数组的维度，对于n行m列的数组，shape为（n，m）
print(ar.size)     # 数组的元素总数，对于n行m列的数组，元素总数为n*m
print(ar.dtype)    # 数组中元素的类型，类似type()（注意了，type()是函数，.dtype是方法）
print(ar.itemsize) # 数组中每个元素的字节大小，int32l类型字节为4，float64的字节为8
```
<br>

## 数组的构造

```python
# 创建数组：arange()，类似range()，在给定间隔内返回均匀间隔的值。
print(np.arange(5.0,12,2))  # 返回5.0-12.0，步长为2
# 创建数组：linspace():返回在间隔[开始，停止]上计算的num个均匀间隔的样本。
ar2 = np.linspace(2.0, 3.0, num=5, endpoint=False)
# 创建数组：zeros()/zeros_like()/ones()/ones_like()
ar2 = np.zeros((2,2), dtype = np.int)
ar4 = np.zeros_like(ar3)
ar7 = np.ones_like(ar3)
# 创建一个正方的N*N的单位矩阵，对角线值为1，其余为0
print(np.eye(5))
```
<br>

### ndarray的数据类型

bool 用一个字节存储的布尔类型（True或False）

inti 由所在平台决定其大小的整数（一般为int32或int64）

int8 一个字节大小，-128 至 127

int16 整数，-32768 至 32767

int32 整数，-2 ** 31 至 2 ** 32 -1

int64 整数，-2 ** 63 至 2 ** 63 - 1

uint8 无符号整数，0 至 255

uint16 无符号整数，0 至 65535

uint32 无符号整数，0 至 2 ** 32 - 1

uint64 无符号整数，0 至 2 ** 64 - 1

float16 半精度浮点数：16位，正负号1位，指数5位，精度10位

float32 单精度浮点数：32位，正负号1位，指数8位，精度23位

float64或float 双精度浮点数：64位，正负号1位，指数11位，精度52位

complex64 复数，分别用两个32位浮点数表示实部和虚部

complex128或complex 复数，分别用两个64位浮点数表示实部和虚部

<br>

## 数组形状

```python
# 数组形状：.T/.reshape()/.resize()
# .T方法：转置，一维数组转置后结果不变
ar1 = np.arange(10)
# 改变数组形状，不改变元素个数
ar3 = ar1.reshape(2,5)     # 用法1：直接将已有数组改变形状             
ar4 = np.zeros((4,6)).reshape(3,8)   # 用法2：生成数组后直接改变形状
ar5 = np.reshape(np.arange(12),(3,4))   # 用法3：参数内添加数组，目标形状

# numpy.resize(a, new_shape)：返回具有指定形状的新数组，如有必要可重复填充所需数量的元素。
ar6 = np.resize(np.arange(5),(3,4))
print(ar6)

# 注意了：.T/.reshape()/.resize()都是生成新的数组！！！
```
<br>

## 数组的复制

```python
# 这里ar1和ar2指向同一个值，所以ar1改变，ar2一起改变
ar1 = np.arange(10)
ar2 = ar1
print(ar2 is ar1)
ar1[2] = 9
# copy方法生成数组及其数据的完整拷贝
ar3 = ar1.copy()
print(ar3 is ar1)
ar1[0] = 9
print(ar1,ar3)
# copy方法生成数组及其数据的完整拷贝
```
<br>

## 数组类型的改变

```python
# 数组类型转换：.astype()
ar1 = np.arange(10,dtype=float)
# a.astype()：转换数组类型
ar2 = ar1.astype(np.int32)
# 注意：养成好习惯，数组类型用np.int32，而不是直接int32
```
<br>

## 数组堆叠

```python
# numpy.hstack(tup)：水平（按行顺序）堆叠数组
a = np.arange(5)    # a为一维数组，5个元素
b = np.arange(5,9) # b为一维数组,4个元素
ar1 = np.hstack((a,b))  # 注意:((a,b))，这里形状可以不一样 ar1=array([0, 1, 2, 3, 4, 5, 6, 7, 8])
# numpy.vstack(tup)：垂直（按列顺序）堆叠数组
a = np.array([[1],[2],[3]])   
b = np.array([['a'],['b'],['c'],['d']])   
ar2 = np.vstack((a,b))  # 这里形状可以不一样
# ar2=array([['1'],
#        ['2'],
#        ['3'],
#        ['a'],
#        ['b'],
#        ['c'],
#        ['d']], dtype='<U11')
# numpy.stack(arrays, axis=0)：沿着新轴连接数组的序列，形状必须一样
a = np.arange(5)    
b = np.arange(5,10)
ar1 = np.stack((a,b),axis = 0)按行
ar2 = np.stack((a,b),axis = 1)按列
# 重点解释axis参数的意思，假设两个数组[1 2 3]和[4 5 6]，shape均为(3,0)
# axis=0：[[1 2 3] [4 5 6]]，shape为(2,3)
# axis=1：[[1 4] [2 5] [3 6]]，shape为(3,2)
```
<br>

## 数组拆分

```python
# numpy.hsplit(ary, indices_or_sections)：将数组水平（逐列）拆分为多个子数组 → 按列拆分
# 输出结果为列表，列表中元素为数组
ar = np.arange(16).reshape(4,4)
ar1 = np.hsplit(ar,2)
# numpy.hsplit(ary, indices_or_sections)：将数组水平（逐列）拆分为多个子数组 → 按列拆分
# 输出结果为列表，列表中元素为数组
ar2 = np.vsplit(ar,4)
print(ar2,type(ar2))
```
<br>

## 数组简单运算

```python
# 数组简单运算

ar = np.arange(6).reshape(2,3)
print(ar + 10)   # 加法
print(ar * 2)   # 乘法
print(1 / (ar+1))  # 除法
print(ar ** 0.5)  # 幂
# 与标量的运算

print(ar.mean())  # 求平均值
print(ar.max())  # 求最大值
print(ar.min())  # 求最小值
print(ar.std())  # 求标准差
print(ar.var())  # 求方差
print(ar.sum(), np.sum(ar,axis = 0))  # 求和，np.sum() → axis为0，按列求和；axis为1，按行求和
print(np.sort(np.array([1,4,3,2,5,6])))  # 排序
# 常用函数
```
<br>

## 数组切片

```python
# 基本索引及切片
# 一维数组索引及切片
print(ar[3:6])
# 二次索引，得到一维数组中的一个值
print(ar[2][1]) 
print(ar[2,2])
# 二维数组切片
print(ar[:2,1:])  # 切片数组中的1,2行、2,3,4列 → 二维数组
# 布尔型索引：以布尔型的矩阵去做筛选
ar = np.arange(12).reshape(3,4)
i = np.array([True,False,True])
print(ar[i,:])
# 索引的赋值
b[7:9] = 200
```
<br>

## 随机数生成

```python
# numpy.random.rand(d0, d1, ..., dn): 生成一个[0,1)之间的随机浮点数或N维浮点数组 均匀分布
# numpy.random.randint(low[, high, size]): 返回随机的整数，位于半开区间 [low, high)。
# numpy.random.random_integers(low[, high, size]): 左闭右闭[low, high]
# numpy.random.randn(d0, d1, …, dn)从标准正态分布中返回一个或多个样本值               sigma * np.random.randn(d0, d1, ..., dn) + mu 构建正态分布
# random_sample([size]) 返回半开区间 [0.0, 1.0)的浮点数 == random.random([size]) == random.rand() == random.ranf() == random.sample()                                 (b - a) * random_sample() + a
# numpy.random.choice(a[, size, replace, p]) p概率，replace不放回

a = np.random.rand(2,3)
b = np.random.randint(5, size=(2, 4))
c = np.random.random_integers(5, size=(3.,2.))
d = 2.5 * np.random.randn(2, 4) + 3
e = np.random.choice(5, 3, replace=False, p=[0.1, 0, 0.3, 0.6, 0])


# numpy.random.randint(low, high=None, size=None, dtype='l')：生成一个整数或N维整数数组
print(np.random.randint(2,6,size=5))
print(np.random.randint(2,6,(2,3)))
```
<br>

## 数组排列

```python
# np.random.shuffle 现场修改序列，改变自身内容，但是只打乱第一维度
>>> arr = np.arange(9).reshape((3, 3))
>>> np.random.shuffle(arr)
>>> arr
array([[3, 4, 5],
       [6, 7, 8],
       [0, 1, 2]])
# np.random.permutation 返回一个随机排列，也是沿着第一个维度
>>> arr = np.arange(9).reshape((3, 3))
>>> np.random.permutation(arr)
array([[6, 7, 8],
       [0, 1, 2],
       [3, 4, 5]])
```

<br>

分布
beta(a, b[, size])

贝塔分布样本，在 [0, 1]内。
binomial(n, p[, size])

二项分布的样本。
chisquare(df[, size])

卡方分布样本。
dirichlet(alpha[, size])

狄利克雷分布样本。
exponential([scale, size])

指数分布
f(dfnum, dfden[, size])

F分布样本。
gamma(shape[, scale, size])

伽马分布
geometric(p[, size])

几何分布
gumbel([loc, scale, size])

耿贝尔分布。
hypergeometric(ngood, nbad, nsample[, size])

超几何分布样本。
laplace([loc, scale, size])

拉普拉斯或双指数分布样本
logistic([loc, scale, size])

Logistic分布样本
lognormal([mean, sigma, size])

对数正态分布
logseries(p[, size])

对数级数分布。
multinomial(n, pvals[, size])

多项分布
multivariate_normal(mean, cov[, size])

多元正态分布。

>>> mean = [0,0]
>>> cov = [[1,0],[0,100]] # diagonal covariance, points lie on x or y-axis
>>> import matplotlib.pyplot as plt
>>> x, y = np.random.multivariate_normal(mean, cov, 5000).T
>>> plt.plot(x, y, ‘x‘); plt.axis(‘equal‘); plt.show()


negative_binomial(n, p[, size])

负二项分布
noncentral_chisquare(df, nonc[, size])

非中心卡方分布
noncentral_f(dfnum, dfden, nonc[, size])

非中心F分布
normal([loc, scale, size])

正态(高斯)分布

Notes

The probability density for the Gaussian distribution is



where  is the mean and  the standard deviation. The square of the standard deviation, , is called the variance.

The function has its peak at the mean, and its “spread” increases with the standard deviation (the function reaches 0.607 times its maximum at  and  [R217]).

 

Examples

Draw samples from the distribution:

>>> mu, sigma = 0, 0.1 # mean and standard deviation
>>> s = np.random.normal(mu, sigma, 1000)
>>> Verify the mean and the variance:

>>> abs(mu - np.mean(s)) < 0.01
>>> True
>>> abs(sigma - np.std(s, ddof=1)) < 0.01
>>> True
>>> Display the histogram of the samples, along with the probability density function:

>>> import matplotlib.pyplot as plt
>>> count, bins, ignored = plt.hist(s, 30, normed=True)
>>> plt.plot(bins, 1/(sigma * np.sqrt(2 * np.pi)) *
>>> ...                np.exp( - (bins - mu)**2 / (2 * sigma**2) ),
>>> ...          linewidth=2, color=‘r‘)
>>> plt.show()


pareto(a[, size])

帕累托（Lomax）分布
poisson([lam, size])

泊松分布
power(a[, size])

Draws samples in [0, 1] from a power distribution with positive exponent a - 1.
rayleigh([scale, size])

Rayleigh 分布
standard_cauchy([size])

标准柯西分布
standard_exponential([size])

标准的指数分布
standard_gamma(shape[, size])

标准伽马分布
standard_normal([size])

标准正态分布 (mean=0, stdev=1).
standard_t(df[, size])

Standard Student’s t distribution with df degrees of freedom.
triangular(left, mode, right[, size])

三角形分布
uniform([low, high, size])

均匀分布
vonmises(mu, kappa[, size])

von Mises分布
wald(mean, scale[, size])

瓦尔德（逆高斯）分布
weibull(a[, size])

Weibull 分布
zipf(a[, size])

齐普夫分布
随机数生成器
RandomState

Container for the Mersenne Twister pseudo-random number generator.
seed([seed])

Seed the generator.
get_state()

Return a tuple representing the internal state of the generator.
set_state(state)

Set the internal state of the generator from a tuple.


## 数据输入和输出

```python
import os
os.chdir('C:/Users/Hjx/Desktop/')
ar = np.random.rand(5,5)
# 存储数组数据 .npy文件
np.save('arraydata.npy', ar)

# 读取数组数据 .npy文件
ar_load =np.load('arraydata.npy')

# 存储/读取文本文件
ar = np.random.rand(5,5)
np.savetxt('array.txt',ar, delimiter=',')
# np.savetxt(fname, X, fmt='%.18e', delimiter=' ', newline='\n', header='', footer='', comments='# ')：存储为文本txt文件
ar_loadtxt = np.loadtxt('array.txt', delimiter=',')
print(ar_loadtxt)
# 也可以直接 np.loadtxt('C:/Users/Hjx/Desktop/array.txt')
```
<br>

## 数组重复

```python
# 重复数组的一部分
>>> np.repeat(3, 4)
array([3, 3, 3, 3])
>>> x = np.array([[1,2],[3,4]])
>>> np.repeat(x, 2)
array([1, 1, 2, 2, 3, 3, 4, 4])
>>> np.repeat(x, 3, axis=1)
array([[1, 1, 1, 2, 2, 2],
       [3, 3, 3, 4, 4, 4]])
>>> np.repeat(x, [1, 2], axis=0)
array([[1, 2],
       [3, 4],
       [3, 4]])
# 创建一个有重复模式的数组
>>> np.tile(a, (2, 1, 2))
array([[[0, 1, 2, 0, 1, 2]],
       [[0, 1, 2, 0, 1, 2]]])
```

<br>

## 多维数组的任意百分比分位数

```python
>>> a = range(1,101)
#求取a数列第90%分位的数值
>>> np.percentile(a, 90)
Out[5]: 90.10000000000001

>>> a = range(101,1,-1)
#百分位是从小到大排列
>>> np.percentile(a, 90)
Out[7]: 91.10000000000001
    
>>> a = np.array([[10, 7, 4], [3, 2, 1]])
>>> np.percentile(a, 50) #50%的分位数，就是a里排序之后的中位数
3.5
>>> np.percentile(a, 50, axis=0) #axis为0，在纵列上求
array([[ 6.5,  4.5,  2.5]])
>>> np.percentile(a, 50, axis=1) #axis为1，在横行上求
array([ 7.,  2.])
>>> np.percentile(a, 50, axis=1, keepdims=True)  #keepdims=True保持维度不变
array([[ 7.],
       [ 2.]])
```

