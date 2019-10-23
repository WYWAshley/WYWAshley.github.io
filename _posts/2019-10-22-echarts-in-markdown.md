---
layout: post
title: Use echarts in markdown 
categories: [visualization]
description: a QA for echarts in markdown
keywords: echarts, markdown
---

## 在 markdown blog 中应用 echarts

通过[知乎的回答](国内在线图表工具，你能说出几个？ - ayura的回答 - 知乎 https://www.zhihu.com/question/38931668/answer/252416334)我了解到了 [echarts](https://www.echartsjs.com/zh/index.html) 这款可视化工具，相比于我之前接触到的一些图表 api 如 python 的 matplotlib，它的文档更加的全面，上手也很快。

具体的细节我就不再赘述，在这里我简要介绍一下，如何在编写 markdown 文档的时候，插入 html 代码，利用 echarts API 来进行图表绘制。

### Download

首先是下载 echarts，官方提供了三种下载方式

* npm 安装，运行 `npm install echarts`
* 下载发布版本的编译产物，或下载源代码
* 选择需要的模块，在线定制下载

具体的操作详见[官方下载页面](https://www.echartsjs.com/zh/download.html)。

这里需要说明的是，第一种方法即用 npm 来下载，会下载源代码，具体文件的格式我不清楚，但一定包含了所有相关的内容。我一开始使用的是第三种，非常的方便，只要选择自己需要的模块，官方会根据你的需求自动化生成一个较为精简的 json 文档，只需要引入该文档即可。

我最后为了后续开发方便，使用了第二种方法，下载了发布版本的编译文档，即一系列的 json 文件。

### Use

之前说到我是为了能够在 markdown 当中添加图表。其实说的不够准确，应该是在 github.io 个人网站上应用。编辑的文档通过 [jekyll](http://jekyllcn.com/) 将 markdown 文档构建为 html 静态网页。

因此，要在用 markdown 写成的 blog 文档添加 echarts 图表，只要插入 html 代码即可。我尝试了一下，在使用 [Typora](https://www.typora.io/) 作为 Markdown 编辑器，用 jekyll 来将 markdown 文档转化为 html 的情景下，可以直接在文档中插入 html 代码，实现 echarts 图表的绘制。

下图即是在文档中插入代码之后绘制的图表，建议直接用 [vscode](https://code.visualstudio.com/) 这类编辑器插入，若是使用标准的 markdown 编辑器，会出现 html 代码外套标签导致无法正常显示图片。

<div id="container" style="weight:80%; height: 600px"></div>
<script type="text/javascript" src="/js/dist/echarts.min.js"></script>
<script type="text/javascript" src="/js/dist/echarts-gl.min.js"></script>
<script type="text/javascript" src="/js/dist/ecStat.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/dataTool.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/bmap.min.js"></script>
<script type="text/javascript">
var dom = document.getElementById("container");
var myChart = echarts.init(dom);
var app = {};
option = null;
option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [120, 200, 150, 80, 70, 110, 130],
        type: 'bar'
    }]
};
if (option && typeof option === "object") {
    myChart.setOption(option, true);
}
</script>

上图的代码如下所示

```html
<div id="container" style="height: 100%"></div>
<script type="text/javascript" src="/js/dist/echarts.min.js"></script>
<script type="text/javascript" src="/js/dist/echarts-gl.min.js"></script>
<script type="text/javascript" src="/js/dist/ecStat.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/dataTool.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/bmap.min.js"></script>
<script type="text/javascript">
var dom = document.getElementById("container");
var myChart = echarts.init(dom);
var app = {};
option = null;
option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [120, 200, 150, 80, 70, 110, 130],
        type: 'bar'
    }]
};
if (option && typeof option === "object") {
    myChart.setOption(option, true);
}
</script>
```

其中，引入的各类 .js 文件，来源于下载的 echarts 编译后的 .js 文件，在下载页面有详细的说明，这之中有一些在软件包中没有的，比如 echarts-gl.min.js 文件，需要自己下载并放到项目的目录中去。

**注意：插入 markdown 文件当中的代码，可以由官方实例的预览网页直接生成，且插入时不需要消除换行符和格式。**

这些代码并非是我自己编写的，而是来源于 echarts 的[官方实例](https://www.echartsjs.com/examples/zh/editor.html?c=bar-negative)，还有许多的实例可以参考，绘制图表的时候只需要在实例上进行改动即可。

若要改动图表的格式、内容等一系列元素，参考官方的[配置项手册](https://www.echartsjs.com/zh/option.html)即可。

### References

1. Echarts 官方网站：https://www.echartsjs.com/zh/index.html
2. Echarts 官方实例：https://www.echartsjs.com/examples/zh/index.html
3. Echarts 配置项手册：https://www.echartsjs.com/zh/option.html
4. Echarts cheat sheet 快速定位配置项：https://www.echartsjs.com/zh/cheat-sheet.html