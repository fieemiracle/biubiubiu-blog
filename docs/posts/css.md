---
date: 2025-09-08
category:
  - CSS
tag:
  - BFC
  - 重绘
  - 重排
---

# CSS（Cascading Style Sheets，层叠样式表）

## 创作背景

  就像调色板能将白纸变为艺术品，CSS能将枯燥的HTML文本变为视觉盛宴。`CSS`是一种样式表语言，用于描述`HTML或XML`（包括`SVG`）文档的呈现方式。
  核心思想：实现内容与表现的分离
  - `HTML`负责定义网页的结构和内容
  - `CSS`负责控制网页的外观和布局
  没有`CSS`，网页将只是枯燥的、堆砌在一起的文本和链接。`CSS`是构建美观、用户体验良好的现代网页的基石。

## BFC（Block Formatting Context）

  区块格式化上下文(`BFC`)是`Web`页面的可视CSS渲染的一部分，是块级盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域
  创建BFC的方式：
  - 文档的根元素（`<html>`）
  - 浮动元素（即`float`值不为`none`的元素）
  - 绝对定位元素（`position`值为`absolute`或`fixed`的元素）
  - 行内快元素（`display`值为`inline-block`的元素）
  - 表格单元格（`display`值为`table-cell`，`HTML`表格单元格默认值）。
  - 表格标题（`display` 值为`table-caption`，`HTML`表格标题默认值）。
  - 匿名表格单元格元素（`display` 值为 `table`（`HTML`表格默认值）、`table-row`（表格行默认值）、`table-row-group`（表格体默认值）、`table-header-group`（表格头部默认值`table-footer-group`（表格尾部默认值）或 `inline-table`）。
  - `overflow` 值不为 `visible` 或 `clip` 的块级元素。
  - `display` 值为 `flow-root` 的元素。
  - `contain` 值为 `layout`、`content` 或 `paint` 的元素。
  - 弹性元素（`display` 值为 `flex` 或 `inline-flex` 元素的直接子元素），如果它们本身既不是弹性、网格也不是表格容器。
  - 网格元素（`display` 值为 `grid` 或 `inline-grid` 元素的直接子元素），如果它们本身既不是弹性、网格也不是表格容器。
  - 多列容器（`column-count` 或 `column-width` 值不为 `auto`，且含有 `column-count`: 1 的元素）。
  - `column-span` 值为 `all` 的元素始终会创建一个新的格式化上下文，即使该元素没有包裹在一个多列容器中（规范变更、`Chrome bug`）
  
  格式化上下文影响布局，一般为定位和清除浮动创建新的`BFC`，而不是更改布局，因为`BFC`能够：

  - 包含内部浮动
  - 排除外部浮动
  - 阻止外边距重叠

## 重排&重绘