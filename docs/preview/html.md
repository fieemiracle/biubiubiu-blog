---
title: HTML
createTime: 2025/10/10 16:52:04
permalink: /article/8efeg1zk/
---

## 一、html元素分类

### 1、按语义分类

1. ==块级元素==

**独占一行，默认宽度为副元素的100%，可设置宽高**

`div`、`p`、`h1~h6`、`ul`、`ol`、`li`、`article`、`section`、`header`、`footer`

2. ==行内元素==

**不独占一行，宽度由内容决定，不可设置宽高（或设置无效）**

`span`、`a`、`strong`、`em`、`img`、`input`、`label`、`br`

1. ==行内块元素==

**兼具行内元素和块级元素特点，不独占一行但可设置宽高**

`button`、`input`、`select`、`textarea`、`img`，部分元素需通过`CSS`指定`display: inline-block`。

:::: warning 为什么`img`既是行内元素，又是行内块元素？

`img` 属于“行内级可替换元素”（inline-level replaced element）。

- 行内级：参与行内格式化上下文，和文字同一行排布，不独占一整行；
- 可替换：有内在尺寸（intrinsic size），并允许通过 `width/height` 设置尺寸，表现出类似 `inline-block` 的可设置宽高与盒模型特性。

因此它既具有行内元素的排版行为，又具备行内块元素可设置尺寸的能力。多数浏览器默认样式为 `display: inline`，但“可替换元素”的特性使其实际表现接近 `inline-block`。

::::

### 2、按功能分类

1. ==结构类元素==

**用于构建页面骨架，体现文档结构**

`html`、`head`、`body`、`header`、`nav`、`main`、`footer`、`section`、`article`

2. ==文本类元素==

**用于展示和格式化文本内容**

`p`、`h1~h6`、`strong`、`em`、`span`、`br`、`hr`

3. ==链接与导航元素==

**用于页面跳转或锚点链接**

`a`、`nav`

4. ==列表元素==

**用于展示有序或无序列表内容**

`ul`、`ol`、`li`、`dl`、`dt`、`dd`

5. ==媒体元素==

**用于嵌入图片、音频、视频等资源**

`img`、`audio`、`video`、`iframe`

6. ==表单元素==

**用于用户输入和交互**

`form`、`input`、`button`、`select`、`textarea`、`label`

7. ==语义化分组元素==

**用于对内容进行逻辑分组，增强可读性和`SEO`**

`div`、`span`、`figure`、`figcaption`


### 3、按是否为空元素分类

1. ==空元素==

**没有闭合标签，也不包含内容，通常用于插入资源或执行命令**

`img`、`br`、`hr`、`input`、`meta`、`link`

2. ==非空元素==

**有开始标签和结束标签，可包含文本或其他元素**

`p`、`div`、`a`

### 4、按是否为替换元素分类

1. ==替换元素==

**内容不由`CSS`控制，由外部资源决定（如图片、表单控件）**

`img`、`input`、`video`、`iframe`

2. ==非替换元素==

**内容由元素本身的文本或子元素决定，受`CSS`样式影响**

`p`、`div`、`span`、`h1~h6`