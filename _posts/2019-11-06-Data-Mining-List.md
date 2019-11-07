---
layout: post
title: Data Mining Knowledge Points
categories: [machine learning]
description: a list of conception in Data Mining
keywords: data mining
---

# Data Ming Exam points



<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
inlineMath: [['$','$']]
}
});
</script>



#### — 带上计算器！

### Chapter 1

- 不同变量的相似性计算
- 统计量的计算和其优缺点
- 学会给定数据手工画 box plot
- 正态分布的几个数值
- 给定数据画直方图
- 直方图和boxplot的比较
- 散点图对于数据相关性的展现
- 数据可视化的一些基本认识 — 看图说话
- 散点图矩阵的概念

### Chapter 2

- 每一种不同类型的数据，相似性如何衡量
- Minkowski distance
- L1 norm, L2 norm
- 余弦相似度
- 基本全部都很重要

### Chapter 3

- chi-square test
- Correlation coefficient
- Covariance, 独立性等关系
- feature extraction
- PCA，LDA 基本的结论
    - PCA - 根据协方差矩阵求出相应的特征值和特征向量，然后按照特征值从大到小的顺序选取特征向量作为变换矩阵的列向量
    - LDA 有点难，具体的步骤可以参考[这里](https://zhuanlan.zhihu.com/p/33742983)
        - 找到 $S_B = S_1 + S_2$ 和 $S_W = (\mu_1 - \mu_2)(\mu_1-\mu_2)^T$
        - $S_B\alpha=\lambda S_W\alpha$ 解出不同的特征向量 $\alpha$
        - 得到转换矩阵 $A=[\alpha_1, ...,\alpha_c]$
- sampling 的方法，strength and weakness
    - simple random sampling
    - sampling without replacement (不放回抽样)
    - sampling with replacement (放回抽样)
    - cluster or stratified sampling (分类抽样，按类别占比选取样本个数 for skewed data)
* Data Transformation 相当重要，mix-max，z-score，decimal scaling
    * Normalization
        * mix-max `v' = (v-min)/(max-min)*(max'-min')+min'`
        * z-score `v' = (v-mu)/sigma`, where mu is mean, sigma is standard varience
        * decimalscaling `v' = v / 10^j`, Where j is the smallest integer such that $Max(\left\lvert{v’}\right\lvert) < 1$ 
    - Discretization
        - 分箱法
            - Equal-width 等宽，然而极端值决定了分箱情况，且对 skew data 的适应性不好
            - Equal-depth 等宽或等频率，每个箱子中有等量的数据，但是管理分类属性会很困难，K-Means 具有更好的分类结果
        - 直方图分析
        - 聚类分析
        - 决策树分析
        - 相关性分析，如 chi-square 测试
- 概念分层
    - 对 data set 中的 attributes 进行分层，如 street < city < state < country

### Chapter 4

- basic concepts
    - Item-set
    - K-itemset
    - (absolute) support / support count
    - (relative) support
    - Frequent

- 关联规则
    - Support and confidence
    - Beer -> Diaper (60%, 100%)
- Closed pattern 和 Max pattern
    - closed pattern - 没有比他大同时 support 和它一样的 pattern
    - max pattern - 没有比它还要大的 frequent pattern
- Apriori - 方法、弱点
    - 若一个 itemset 不是 frequent，那么它的 superset 也一定不是 frequent，因而不需要被测试
    - 层层递进，一直到 length k frequent item set
    - 两个大部分，一个是层层递进的 generate item set，仅保留 frequent ones 然后再次组合，直到 item set 的元素数目为 threshold+1 时，进行剪枝，即剔除子集中包含非 frequent 的 item set。最后再扫描一遍数据库确认。
- FP-Tree - 构造、给定导出频繁模式
    - 扫描一遍 data base 得到每个 pattern 的频率，根据这个频率排序，然后再把原来 data base 里面的 item 按照 pattern frequency 排序，最后按照这个来画 FP-Tree
    - 对 FP-Tree 进行 mining 的时候，按照之前的到 frequency 的倒序，对每个 item 先找到 conditional pattern Base 也就是前缀，再找到出现次数大于等于 min support 的 item，再进行组合得到 frequent pattern
    - 优点：完整性和简洁性
- Sequential pattern
    - apriori -> GSP
- Pattern evaluation - Interestingness Measure Correlations ppt slides
    - Lift = P(AUB) / P(A)P(B)

### Chapter 5

* Classification
    * Model construction
    * Model usage
        * estimate accuracy
        * test set / validation set (latter used to select model)

- Decision Tree *

    - 自顶向下的递归分治法
    - 属性已经分类（若是连续型的值就提前离散化）
    - 根据选择的属性将 examples 分组
    - information entropy $H(x)=-\Sigma_xp(x)log_2p(x)$
        - entropy 越大，则不确定性越大，该信息量越大
    - 在决策树构建时，选择获得信息量最大（highest **information gain**）的 attribute - ID3
        - $Info(D)=-\sum_{i=1}^m p_ilog_2(p_i)$ 选择 attribute 之前的信息熵
        - $Info_A(D)=\sum_{j=1}^v \frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert}\times Info(D_j)$ 选择attribute 之后的信息熵
        - $Gain(A)=Info(D)-Info_A(D)$ 二者之差为信息增量
    - GainRatio to overcome attributes with large number of values
    - 对于**连续的属性**，需要进行划分，一般选取中间值，但依据始终是选取信息量要求最小的点
    - 当 attribute 的值特别多的时候，上面公式得到的 gain 往往不能直接参考，需要改动，引入了 **Gain Ratio** 的概念，称之为 C4.5，合起开就是 ID3/C4.5
        - $SplitInfo_A(D)=-\sum_{j=1}^{v} \frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert}\times log_2(\frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert})$
        - 也就是说，最后选择作为 splitting attribute 的属性，**一定要是 Gain Ratio 最大的**
    - **Gini Index**
        - $gini(D)=1-\sum_{j=1}^{n}p_j^2$
        - $gini_A(D)=\frac{\left\lvert{D_1}\right\lvert}{\left\lvert{D}\right\lvert}gini(D_1)+\frac{\left\lvert{D_2}\right\lvert}{\left\lvert{D}\right\lvert}gini(D_2)$
        - $\Delta gini(A)=gini(D)-gini_A(D)$
        - 取 $\Delta$ 最小的分类方式
    - Comparation
        - Information gain
            - 多值 attributes 的时候会有偏差
        - Gain ratio
            - 倾向于不平衡的分组，导致会有一个分组比其他小的多
        - Gini Index
            - 多值 attributes 的时候会有偏差
            - Classes 较大的时候计算困难
            - 倾向于平衡的分组
    - Overfitting and Tree Pruning
        - too many branches and poor accracy for unseen samples
        - prepruning (按照阈值停止构建决策树) and postpruning (按照某种方法剪枝)

- Bayes Classification *

    - 贝叶斯概率 $P(B)=\sum_{i=1}^MP(B\lvert A_i)P(A_i)$

    - 贝叶斯理论 $P(H\lvert X)=\frac{P(X\lvert H)P(H)}{P(X)}$

        - **X** 为一个 class label 未知的 data sample
        - **H** 是 **X** 属于 Class C 的假设
        - 贝叶斯公式的作用是将“已知 **X** 求 **X** 属于 Class C”的问题转化为求“已知有属于 Class C 的若干实例，这些实例中 **X** 出现的频率”，化未知为已知
        - 其实就是后验概率和先验概率的转化，posteriori = likehood x prior / evidence

    - 计算所有 $P(C_i\lvert X)$，**选取最大结果的 $C_i$ 作为 X 的分类结果**

    - <u>***注意！！！由于 P(X) 的概率在比较时是一致的，因而考虑的就只有 $P(X\lvert C_i)$ 和 $P(C_i)$ 的乘积，因而两个都不能漏掉！！！***</u>

    - 然而这需要事先知道很多值

    - 朴素贝叶斯 Naive Bayes Classifier

        - 建立在每个 attribute 都互相独立，即 $P(X\lvert C_i)=\prod_{k=1}^{n}P(x_k\lvert C_i)$

        - 大大减小了计算开销

        - 若属性 k 是离散的，即分类别的，那么$P(x_k\lvert C_i)$就是$C_i$中$x_k$出现的频率

        - 若属性 k 是连续的，那就用高斯分布区计算

            $$P(x_k\lvert C_i) = g(x_k,\mu_{C_i},\sigma_{C_i}) = \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$$

        - 因为是连乘，所以所有属性的概率都不能为 0，因此可以使用 Lappacian correction

            - 给每个 case 加 1
            - 修正过的概率近似于未修正的概率

        - advantage

            - 简单，且效果不错

        - disadvantage

            - 假设是所有 class 相互独立，因而准确性降低
            - 属性之间可能存在依赖性

- Model Evaluation and Selection 所有方法和概念

    - Confusion Matrix 和后续的计算方法
        - True Positives 分对了，False Positives 错的被认为是对的，False Negatives 对的被认为是错的，True Negatives 错的分对了
        - Accuracy = (TP + TN) / ALL
        - Error rate = (FP + FN) / ALL
        - Sensitivity = TP / P, P = TP + FN
        - Specificity = TN / N, N = FP + TN
        - Precision:exactness **找到的分类有多少占比是找对了** precision = TP / (TP + FP)
        - Recall:completeness **找到了对的分类占对的分类的总数** recall = TP / (TP + FN) 就是 sensitivity
        - precision 越高，recall 越低，反之亦然
        - F measure $F = \frac{2\times precision \times recall}{precision + recall}$ 调和平均数
        - $F_\beta = \frac{(1+\beta^2)\times precision \times recall}{\beta^2 \times precision + recall}$ Weighted measure，$\beta$ 代表 recall 参数权重的倍数

- Holdout method、cross-validation 等计算精确度的

    - Holdout method
        - 数据被随机分成了 2 部分，2/3 训练，1/3 测试
        - Random sampling - 重复 k 次 holdout 然后取平均值
    - Cross-validation
        - 把数据集划分为大小相同的 k 个不相交子集
        - 第 i 次迭代的时候，用 $D_i$ 作为测试集，其他子集作为训练集
        - Leave-one-out，k 为数据集的总数
        - Stratified cross-validation，folds 分层
    - Bootstrap
        - 对小型数据集适应很好
        - 即每次训练的样本是放回抽样的
        - 最通用的是 .632 bootstrap
            - 被抽中的大概在 0.632，剩下的作为测试集
            - 循环 k 次，计算 accuracy

- test of statistical significance 显著性检验 和 confidence limits 置信区间

    - t-test

- ROC 曲线的理解

    - TP-Rate，即找到的真的 A 类占 A 类总数的比例，为 100% 是最好的
    - FP-Rate，即找到的假的 A 类占非 A 类总数的比例，为 0 是最好的
    - ROC 曲线越凸越好

- 碰到不均衡的数据如何做 oversampling等

    - Oversampling 对少的分类的数据重新采样
    - Under-sampling 对多的分类的样本随机删除
    - Threshold-moving 修改决策权重

### Chapter 6

* 聚类的基本概念
    * Cluster 组内元素相似度高，组间元素相似度低
    * Cluster analysis 找到数据之间的相似性然后归类分组
    * Unsupervised learning 没有预定义的类别
    * dissimilarity/similarity metric
* 主要的聚类方法
    * Partitioning approach - k-means, k-medoids
    * Hierarchical approach
    * Density-based approach 
    * Grid-based approach
    * Model-based
    * Frequent pattern-based
    * User-guided or constraint-based
    * Link-based clustering

- Partitioning Method
    - $E=\sum_{i=1}^{k}\sum_{p\in C_i}(p-c_i)^2$
    - 使得 $E$ 最小，即每个类别的点都要离中心点足够近
- K-Means *
    - 每个 cluster 由中心点来表征
    - 在找到这样一组 cluster，使得每个 cluster 内的数据点到中心点的距离之和最小
    - $min_{r_{nk},\mu_k}\sum_{n=1}^{N}\sum_{k=1}^{K}r_{nk}\left\lvert\left\lvert{x_n-\mu_k}\right\lvert\right\lvert$，目标函数，其中 $r_{nk}$ 仅当 $x_n$ 属于 $k$ 号聚类的时候才为 1，否则为 0。这个目标函数计算出的就是各聚类的点到中心点的欧式距离之和
    - 方法
        - 第一步随机划分 $K$ 个聚类
        - 第二步保持各个点所属聚类不变，通过改变中心点的位置使得目标函数最小化
            - 保持 $r_{nk}$ 不变对 $\mu_k$ 求导
            - $\mu_k=\frac{\sum_nr_{nk}x_n}{\sum_nr_{nk}}$
            - 上式含义即**求每个聚类当中的中心点也就是均值**，因此才叫 K-Means
        - 然后再重新调整各个点所属的 cluster 使得目标函数最小，即保持中心点不变，通过改变各个点所属聚类 $r_{nk}$ 的值来使得目标函数最小化
        - 再执行第二步，求均值，循环直到聚类分配 $r_{nk}$ 不再改变
    - Strength
        - 有效，复杂度为 O(tkn)，n 是对象数目，k 是聚类数目，t 是迭代次数
    - 结论
        - 在局部最优的情况下终止循环
    - 缺点 
        - 只针对连续的 N 维空间
            - 对于分类的属性，用 k-modes 方法
            - 相比而言 k-medoids 有更为广泛的应用对象
        - 必须提前定好聚类的数量 K
        - 对噪声和异常值很敏感
        - 由于应用的是欧式距离，因此只能发现**“凸”**形状的聚类
    - 变体
        - 大多数集中在
            - 最初 k 个聚类的选择
            - 差异性的计算
            - 均值的计算方法
        - 对分类属性的解决方案 - k-modes
        - 分类+连续数据混合的方案 - k-prototype
- K-Medoids *
    - 每个 cluster 由组内的一个数据来表征
    - 主要步骤
        - 任意选择 k 个初始的节点作为 intial medoids
        - 将剩下的节点分配给距离最近的 medoids
        - 在 cost 减少的情况下
            - 对每个中心点，找一个非中心点的节点和它对换，再次计算 cost
            - 如果交换之后的 cost 比原来的大，那么 undo swap
            - 如果交换之后的 cost 比原来的小，那么 save swap
    - PAM (Partitioning Around Medoids) - 即上述方法，对较小的数据集应用较好，数据集大的时候由于计算复杂度上升，效果一般
    - 改进版 - VLARA (PAM on samples) / CLARANS (Randomized re-sampling)
- 高维度的聚类
    - subspace-clustering
    - dimensionality reduction approaches
- spectual clusting
- evaluation