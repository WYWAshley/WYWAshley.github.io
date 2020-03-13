<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
inlineMath: [['$','$']]
}
});
</script>

layout: post
title: 盒子放球类问题总结（斯特林数）
categories: [mathematics]
description: 关于盒子中放球类问题的总结说明，并介绍了斯特林数
keywords: 斯特林数, 统计学

2020年美团数据分析笔试题第10道

## 引入

一个正整数N可以分解为M(M>1)个正整数的和，即N=K+L，例如N=5、M=2时可以分解为(1+4,2+3)。

给定一个正整数N(1<N<200)及正整数M(1<M<200)，求有多少种可能的分解组合(注：K+L和L+K算一种)

如：输入6，3，输出3.

```python
tmp = input().split(',')
n = int(tmp[0])
m = int(tmp[1])

if n>=m:
    # 因为要求非空，所以先在每个盒子里放进一球
    n = n-m
    dp = [[0 for _ in range(m+1)] for _ in range(n+1)]
    for i in range(0, n+1):
        for j in range(1, m+1):
            # 边界条件，当没有球或者只有一个球或者只有一个盒子时，返回1
            if i==0 or i==1 or j==1:
                dp[i][j] = 1
            # 当球少于盒子，只能跳过这个盒子
            elif i < j:
                dp[i][j] = dp[i][j-1]
            # 否则可以选择跳过这个盒子或者每个盒子都再放一个球
            else:
                dp[i][j] = dp[i][j-1]+dp[i-j][j]
    print(dp[n][m])

else:
    print(0)
```

总结一下盒子中放球的问题。

### 1. 球相同，盒子不同，无空箱

​			此时相当于是插板法，n个球中间有n-1个间隙，分成m个盒子就是选出m-1个间隙。
$$
f = \begin{cases}
0 & n<m \\
{C_{n-1}^{m-1}} & n>=m
\end{cases}
$$

### 2. 球相同，盒子相同，可以空箱

​			和第1种情况一起讨论，没有空箱存在相当于我们可以先假设在每个盒子中都已经放入1个球，那么相		当于此时的n是**n+m**，再代入上式

### 3. 球不同，盒子相同，无空箱

​			这种情况就是第二类斯特林数。对于n个球，如果前面的n-1个已经放在了m个箱子里，那么现在第n	个球放在那个箱子都是可以的，然后又因为球之间是不同的，所以有m*dp\[n-1\]\[m\]；如果前面n-1个球已	经放在了m-1个箱子里，那么现在第n个球必须要新开一个箱子来存放，所以dp\[n-1]\[m-1]。
$$
dp[n][m] = \begin{cases}
m*dp[n-1][m]+dp[n-1][m-1] & 1\le m<n \\
1 & n=m\ge0 \\
0 & m=0
\end{cases}
$$

### 4. 球不同，盒子相同，允许空箱

​			根据第3种情况推进，允许空箱，并且箱子之间都相同，就意味着m个箱子，我可以空出0个、1个、2	个......那么$ans = \sum_{i=0}^mdp[n][m]$ 其中的dp\[n]\[m]是上述第3种情况。

### 5. 球不同，盒子不同，无空箱

​			相当于是第3种情况的盒子从相同变成了不同，那么就是再个m个盒子来个全排列就可以了。$ans=dp[n][m]*A_{m}^m$ 其中这里的dp\[n]\[m]是指第3种情况。

### 6. 球不同，盒子不同，允许空箱

​			这个最简单了，相当于每个小球都可以随意选择m个箱子，$ans=m^n$。

### 7. 球相同，盒子相同，允许空箱

​		现在有n个球，和m个箱子，我可以选择在所有箱子里面都放上1个球，也可以不选择这个操作。

​		如果选择了这个操作，那么就从dp\[n-m][m]转移过来

​		如果没有选择这个操作，那么就从dp\[n][m-1]转移过来
$$
dp[n][m]= \begin{cases}
1 & m=1 \or n=1 \or n=0 \\
dp[n][m-1]+dp[n-m][m] & n\ge m \\
dp[n][m-1] & n<m
\end{cases}
$$

### 8. 球相同，盒子相同，无空箱

​		这个就是上面那道题的考点，因为要求没有空箱，所以事先先放进一个球，也就是开始条件由n个球变为n-m个球。



## 斯特林数

### 1. 第一类斯特林数

形如$\left[\begin{matrix}n\\m\end{matrix}\right]$，也写作 $s(n,k)$。表示把n个数分成k组，每组是一个环，求分成的方案数。每组是一个环意味着怎么转都是一样的，如1,2,3,4和4,1,2,3只算是一种方案。$s(n+1,k)=s(n,k-1)+s(n,k)*n$，意思是最后一个数要么自成一个环，要么加入其他k个环。边界条件$\left[\begin{matrix}0\\0\end{matrix}\right] = 1$。

性质：$s(n,1)=(n-1)!$  ;      $s(n,2)=(n-1)!*\sum_{i=1}^{n-1}\frac{1}{i}$ ;       $\sum_{i=0}^{n}s(n,k)=n!$ ;

证明如下：(https://www.cnblogs.com/ezoiLZH/p/9424911.html)

### 2. 第二类斯特林数

形如$\left\{ \begin{matrix}n\\k\end{matrix}\right\}$，也写作$S(n,k)$，表示把n个数分成k组，组内无序（与第一类的区别在于此，组内没有分别）。$S(n,k)=S(n-1,k-1)+S(n-1,k)*k$，意思是最后一个数要么自成一组，要么加入其它的k组，可以插入k个组。边界条件$\left\{\begin{matrix}0\\0\end{matrix}\right\} = 1$。