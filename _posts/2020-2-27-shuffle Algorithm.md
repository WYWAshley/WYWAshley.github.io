---
layout: post
title: Shuffle Algorithm
categories: [Algorithm]
description: About Shuffling Algorithm
keywords: Shuffle, Algorithm
---

洗牌算法
       由抽牌、换牌和插牌衍生出三种洗牌算法，以及蓄水池抽样问题。

## 一、Fisher-Yates Shuffle算法

> ​        最早提出这个洗牌方法的是 Ronald A. Fisher 和 Frank Yates，即 Fisher–Yates Shuffle，其基本思想就是从原始数组中随机取一个之前没取过的数字到新的数组中，具体如下：
>
> 1. 初始化原始数组和新数组，原始数组长度为n(已知)；
>
> 2. 从还没处理的数组（假如还剩k个）中，随机产生一个[0, k)之间的数字p（假设数组从0开始）；
>
> 3. 从剩下的k个数中把第p个数取出；
>
> 4. 重复步骤2和3直到数字全部取完；
>
> 5. 从步骤3取出的数字序列便是一个打乱了的数列。

   &emsp;&emsp;下面证明其随机性，即每个元素被放置在新数组中的第i个位置是1/n（假设数组大小是n）。 

证明：一个元素m被放入第i个位置的概率P = 前i-1个位置选择元素时没有选中m的概率 * 第i个位置选中m的概率，即 $P=\frac{n-1}{n}\times\frac{n-2}{n-1}\times...\times\frac{n-i+1}{n-i+2}\times\frac{1}{n-i+1}=\frac{1}{n}$                                         

```java
#define N 10
#define M 5
void Fisher_Yates_Shuffle(vector<int>& arr,vector<int>& res)
{
     srand((unsigned)time(NULL));
     int k;
     for (int i=0;i<M;++i)
     {
     	k=rand()%arr.size();
     	res.push_back(arr[k]);
     	arr.erase(arr.begin()+k);
     }
}
```

​       时间复杂度为O(n*n),空间复杂度为O(n)。

<br/>

## 二、Knuth-Durstenfeld Shuffle  

> &emsp;&emsp;Knuth 和 Durstenfeld  在Fisher 等人的基础上对算法进行了改进，在原始数组上对数字进行交互，省去了额外O(n)的空间。该算法的基本思想和 Fisher 类似，每次从未处理的数据中随机取出一个数字，然后把该数字放在数组的尾部，即数组尾部存放的是已经处理过的数字。
>
> &emsp;&emsp;算法步骤为：
>
> 1. 建立一个数组大小为 n 的数组 arr，分别存放 1 到 n 的数值；
>
>    2. 生成一个从 0 到 n - 1 的随机数 x；
>    3. x 的数值，即为第一个随机数；
>    4. 将 arr 的尾元素和下标为 x 的元素互换；
>    5. 到 n - 2 的随机数 x；
>    6. 输出 arr 下标为 x 的数值，为第二个随机数；
>    7. 2个元素和下标为 x 的元素互换；
>          ……
>          如上，直到输出 m 个数为止

该算法是经典洗牌算法。它的证明如下：

对于arr[i],洗牌后在第n-1个位置的概率是1/n（第一次交换的随机数为i）
在n-2个位置概率是[(n-1)/n] * [1/(n-1)] = 1/n，（第一次交换的随机数不为i，第二次为arr[i]所在的位置（注意，若i=n-1，第一交换arr[n-1]会被换到一个随机的位置））
在第n-k个位置的概率是[(n-1)/n] * [(n-2)/(n-1)] *...* [(n-k+1)/(n-k+2)] *[1/(n-k+1)] = 1/n
（第一个随机数不要为i，第二次不为arr[i]所在的位置(随着交换有可能会变)……第n-k次为arr[i]所在的位置）。

```java
void Knuth_Durstenfeld_Shuffle(vector<int>&arr)
{
    for (int i=arr.size()-1;i>=0;--i)
    {
        srand((unsigned)time(NULL));
        swap(arr[rand()%(i+1)],arr[i]);
    }
} 
```

​    时间复杂度为O(n),空间复杂度为O(1),缺点必须知道数组长度n.

原始数组被修改了，这是一个原地打乱顺序的算法，算法时间复杂度也从Fisher算法的 O(n2)提升到了O(n)。由于是从后往前扫描，无法处理不知道长度或动态增长的数组。

<br/>

## 三、Inside-Out Algorithm

>    Knuth-Durstenfeld Shuffle 是一个内部打乱的算法，算法完成后原始数据被直接打乱，尽管这个方法可以节省空间，但在有些应用中可能需要保留原始数据，所以需要另外开辟一个数组来存储生成的新序列。
>     Inside-Out Algorithm 算法的基本思思是从前向后扫描数据，把位置i的数据随机插入到前i个（包括第i个）位置中（假设为k），这个操作是在新数组中进行，然后把原始数据中位置k的数字替换新数组位置i的数字。其实效果相当于新数组中位置k和位置i的数字进行交互。
>
>    如果知道arr的lengh的话，可以改为for循环，由于是从前往后遍历，所以可以应对arr[]数目未知的情况，或者arr[]是一个动态增加的情况。

证明如下：

原数组的第 i 个元素（随机到的数）在新数组的前 i 个位置的概率都是：(1/i) * [i/(i+1)] * [(i+1)/(i+2)] *...* [(n-1)/n] = 1/n，（即第i次刚好随机放到了该位置，在后面的n-i 次选择中该数字不被选中）。
原数组的第 i 个元素（随机到的数）在新数组的 i+1 （包括i + 1）以后的位置（假设是第k个位置）的概率是：(1/k) * [k/(k+1)] * [(k+1)/(k+2)] *...* [(n-1)/n] = 1/n（即第k次刚好随机放到了该位置，在后面的n-k次选择中该数字不被选中）。         

```java
void Inside_Out_Shuffle(const vector<int>&arr,vector<int>& res)
{
	res.assign(arr.size(),0);
	copy(arr.begin(),arr.end(),res.begin());
	int k;
	for (int i=0;i<arr.size();++i)
	{
		srand((unsigned)time(NULL));
		k=rand()%(i+1);
		res[i]=res[k];
		res[k]=arr[i];
	}
} 
```

​    时间复杂度为O(n),空间复杂度为O(n)。

<br/>

## 四、蓄水池抽样

&emsp;&emsp;从N个元素中随机等概率取出k个元素，N长度未知。它能够在o（n）时间内对n个数据进行等概率随机抽取。如果数据集合的量特别大或者还在增长（相当于未知数据集合总量），该算法依然可以等概率抽样。
&emsp;&emsp;算法思路大致如下：

1. 如果接收的数据量小于m，则依次放入蓄水池。
2. 当接收到第i个数据时，i >= m，在[0, i]范围内取以随机数d，若d的落在[0, m-1]范围内，则用接收到的第i个数据替换蓄水池中的第d个数据。
3. 重复步骤2。

算法的精妙之处在于：**当处理完所有的数据时，蓄水池中的每个数据都是以m/N的概率获得的。**

下面用白话文推导验证该算法。假设数据开始编号为1.

**第i个接收到的数据最后能够留在蓄水池中的概率**=**第i个数据进入过蓄水池的概率*****之后第i个数据不被替换的概率**（第i+1到第N次处理数据都不会被替换）。

1. 当i<=m时，数据直接放进蓄水池，所以**第i个数据进入过蓄水池的概率=1**。
2. 当i>m时，在[1,i]内选取随机数d，如果d<=m，则使用第i个数据替换蓄水池中第d个数据，因此**第i个数据进入过蓄水池的概率=m/i**。
3. 当i<=m时，程序从接收到第m+1个数据时开始执行替换操作，第m+1次处理会替换池中数据的为m/(m+1)，会替换掉第i个数据的概率为1/m，则第m+1次处理替换掉第i个数据的概率为(m/(m+1))*(1/m)=1/(m+1)，不被替换的概率为1-1/(m+1)=m/(m+1)。依次，第m+2次处理不替换掉第i个数据概率为(m+1)/(m+2)...第N次处理不替换掉第i个数据的概率为(N-1)/N。所以，之后**第i个数据不被替换的概率=m/(m+1)\*(m+1)/(m+2)\*...\*(N-1)/N=m/N**。
4. 当i>m时，程序从接收到第i+1个数据时开始有可能替换第i个数据。则参考上述第3点，**之后第i个数据不被替换的概率=i/N**。
5. 结合第1点和第3点可知，当i<=m时，第i个接收到的数据最后留在蓄水池中的概率=1*m/N=m/N。结合第2点和第4点可知，当i>m时，第i个接收到的数据留在蓄水池中的概率=m/i*i/N=m/N。综上可知，**每个数据最后被选中留在蓄水池中的概率为m/N**。

这个算法建立在统计学基础上，很巧妙地获得了“m/N”这个概率。

<br/>

* #### 深入一些——分布式蓄水池抽样（Distributed/Parallel Reservoir Sampling）

  一块CPU的计算能力再强，也总有内存和磁盘IO拖他的后腿。因此为提高数据吞吐量，分布式的硬件搭配软件是现在的主流。

  如果遇到超大的数据量，即使是O(N)的时间复杂度，蓄水池抽样程序完成抽样任务也将耗时很久。因此分布式的蓄水池抽样算法应运而生。运作原理如下：

  1. 假设有K台机器，将大数据集分成K个数据流，每台机器使用单机版蓄水池抽样处理一个数据流，抽样m个数据，并最后记录处理的数据量为N1, N2, ..., Nk, ..., NK(假设m<Nk)。N1+N2+...+NK=N。
  2. 取[1, N]一个随机数d，若d<N1，则在第一台机器的蓄水池中等概率不放回地（1/m）选取一个数据；若N1<=d<(N1+N2)，则在第二台机器的蓄水池中等概率不放回地选取一个数据；一次类推，重复m次，则最终从N大数据集中选出m个数据。

  m/N的概率验证如下：

  1. 第k台机器中的蓄水池数据被选取的概率为m/Nk。
  2. 从第k台机器的蓄水池中选取一个数据放进最终蓄水池的概率为Nk/N。
  3. 第k台机器蓄水池的一个数据被选中的概率为1/m。（不放回选取时等概率的）
  4. 重复m次选取，则每个数据被选中的概率为**m\*(m/Nk\*Nk/N\*1/m)=m/N**。
