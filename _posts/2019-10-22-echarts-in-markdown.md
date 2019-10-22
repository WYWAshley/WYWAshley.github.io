---
layout: post
title: Use echarts in markdown 
categories: [visualization]
description: a QA for echarts in markdown
keywords: echarts, markdown
---

## 在 Markdown blog 中应用 echarts

通过[知乎的回答](国内在线图表工具，你能说出几个？ - ayura的回答 - 知乎 https://www.zhihu.com/question/38931668/answer/252416334)我了解到了 [echart](https://www.echartsjs.com/zh/index.html) 这款可视化工具，相比于我之前接触到的一些图表 api 如 python 的 matplotlib，它的文档更加的全面，上手也很快。

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

因此，要在用 markdown 写成的 blog 文档添加 echart 图表，只要插入 html 代码即可。