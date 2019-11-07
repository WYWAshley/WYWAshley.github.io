---
layout: post
title: display latex formula in markdown 
categories: [markdown]
description: a QA for latex formula in markdown
keywords: latex formula, markdown
---

## 在 markdown blog 中正常显示 latex formula

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
inlineMath: [['$','$']]
}
});
</script>



为了在我的 blog 当中正常显示 [LaTeX](https://www.latex-project.org/) 语法的公式，如

$$SplitInfo_A(D)=-\sum_{j=1}^{v} \frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert}\times log_2(\frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert})$$

对应的 LaTeX 代码如下

```latex
$$SplitInfo_A(D)=-\sum_{j=1}^{v} \frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert}\times log_2(\frac{\left\lvert{D_j}\right\lvert}{\left\lvert{D}\right\lvert})$$
```

具体的语法请参照[有关博客](https://www.jianshu.com/p/97ec8a3739f6)。

然而 markdown 是无法解析 LaTeX 公式的，同样基于 jekyll 的 github pages 也无法解析，这个时候只需要在每个 .md 文件的开头加上一段代码即可。

```html
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
inlineMath: [['$','$']]
}
});
</script>
```

加上去之后通过动态的 javascript 脚本让公式能够正常显示。

### 注意！！

用 `\left\lvert` 和 `\right\lvert` 替代绝对值的左右竖线，否则会产生格式错误。因为简单的竖线在 markdown 中是用来表示表格的。