[TOC]

# 快捷键

* `ctrl+/`：源代码模式切换
* `ctrl+shift+L`：侧边栏切换
* `ctrl+shift+1/2/3`：分别打开Outline、Articles、File Tree侧边栏
* `ctrl+T`：创建表格
* `ctrl+B`：加粗

* `ctrl+shfit+F`: 全局查找

# 自定义CSS样式

## 目标

一级标题序号不显示, 之后显示标题序号, 如下图所示

> 一级标题很关键, 运用场景很多, 所以将设置交给用户.
>
> 如, 此文章中写日记, 标题是日期, 此时并不需要序号

## 实现

含标题的三块区域定位:

* 正文: `#write`
* `[TOC]`目录树: `.md-toc-content` 

* 侧边大纲: `.outline-content`