---
layout: post
title: 样条插值的定义
categories: [Stastics]
description: 样条插值的定义
keywords: interpolation, stastics
---
介绍了如何样条插值。

## 样条插值的定义
函数若处处可导，即每一处的左导数和右导数均存在且相等，那么这条函数就是光滑不间断的曲线。

样条插值就是根据每两个相邻的数据点确定一段函数，然后再结合成一个函数，那么就是光滑的函数了。

样条计算模式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213201759236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkxOTYzOQ==,size_16,color_FFFFFF,t_70)
比较不同阶数区别
```python
x = np.linspace(-2*np.pi, 2*np.pi, num=10)
f = lambda x: np.sin(x) + 0.5 * x
x1 = np.linspace(-2*np.pi, 2*np.pi, num=50)
plt.plot(x1, f(x1), label='True Line')

tck1 = spi.splrep(x, f(x), k=1)
tck2 = spi.splrep(x, f(x), k=2)
tck3 = spi.splrep(x, f(x), k=3)
tck4 = spi.splrep(x, f(x), k=4)
x2 = np.linspace(-2*np.pi, 2*np.pi, num=50)
iy1 = spi.splev(x2, tck1)
iy2 = spi.splev(x2, tck2)
iy3 = spi.splev(x2, tck3)
iy4 = spi.splev(x2, tck4)
plt.scatter(x2, iy1, color='r', label='First Order')
plt.scatter(x2, iy2, color='g', label='Second Order')
plt.scatter(x2, iy3, color='brown', label='Third Order')
plt.scatter(x2, iy4, color='brown', label='Fourth Order')
plt.title('Interpolation')
plt.legend()

print('————一阶拟合情况————')
print(np.allclose(f(x2), iy1))
print(np.sum((iy1-f(x2))**2)/len(x2))
print('————二阶拟合情况————')
print(np.allclose(f(x2), iy2))
print(np.sum((iy2-f(x2))**2)/len(x2))
print('————三阶拟合情况————')
print(np.allclose(f(x2), iy3))
print(np.sum((iy3-f(x2))**2)/len(x2))
print('————四阶拟合情况————')
print(np.allclose(f(x2), iy4))
print(np.sum((iy4-f(x2))**2)/len(x2))
```
————一阶拟合情况————
False
0.014289167794024626
————二阶拟合情况————
False
0.0010398929329815821
————三阶拟合情况————
False
0.0013635194627073393
————四阶拟合情况————
False
0.00020543080400768976
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213202402757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkxOTYzOQ==,size_16,color_FFFFFF,t_70)

注意，插值好像只支持内插

```python
x = np.linspace(-2*np.pi, 2*np.pi, num=100)
f = lambda x: np.sin(x) + 0.5 * x
x1 = np.linspace(-2*np.pi, 2*np.pi, num=50)
plt.plot(x1, f(x1), label='True Line')

tck1 = spi.splrep(x, f(x), k=1)
tck2 = spi.splrep(x, f(x), k=2)
x2 = np.linspace(-2*np.pi, -np.pi, num=50, endpoint=False)
x3 = np.linspace(np.pi, 2*np.pi, num=50, endpoint=False)
iy1 = spi.splev(x2, tck1)
iy2 = spi.splev(x3, tck2)

plt.scatter(x2, iy1, color='r', label='First Order')
plt.scatter(x3, iy2, color='g', label='Second Order')
plt.title('Interpolation')
plt.legend()
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213202448853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkxOTYzOQ==,size_16,color_FFFFFF,t_70)
