---
layout: post
title: Markdown用法探索
tagline: Markdown syntax
description: Markdown的一些语法
tags:
  - Markdown
  - 语法
  - 简介
published: true
---
# 一、Markdown简介
<br/>
**Markdown**是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档，然后转换成格式丰富的HTML页面。
											 —— [维基百科](https://zh.wikipedia.org/wiki/Markdown)
    
    
特点：**轻量**、**简单**、**易用**
    
    
开发者：**John Gruber**

Markdown的语法简洁明了、学习容易，而且功能比纯文本更强，因此有很多人用它写博客。世界上最流行的博客平台WordPress和大型CMS如Joomla、Drupal都能很好的支持Markdown。完全采用Markdown编辑器的博客平台有Ghost和Typecho。

用于编写说明文档，并且以“README.MD”的文件名保存在软件的目录下面。

除此之外，现在由于我们有了RStudio这样的神级编辑器，我们还可以快速将Markdown转化为演讲PPT、Word产品文档、LaTex论文甚至是用非常少量的代码完成最小可用原型。在数据科学领域，Markdown已经被确立为科学研究规范，极大地推进了动态可重复性研究的历史进程。[2] 

<br/>

# 二、Markdown语法
<br/>
## 1、标题

<p> 标题能显示出文章的结构。行首插入1-6个 # ，每增加一个 # 表示更深入层次的内容，对应到标题的深度由 1-6 阶。  </p>

> H1: # header1
# H1
H2: ## header2
## H2
H3: ### header3
### H3
H4: #### header4
#### H4
H5: ##### header5
##### H5
H6 ###### header6
###### H6

<br/>

## 2.文本样式
链接 :[Title](URL)
[点击跳转百度](http://www.baidu.com)

加粗 :**Bold**
**加粗效果**

斜体字 :*Italics*
*斜体效果*

*删除线 :~~text~~
~~删除线效果~~

**有序列表 : '数字' + '.'**
1. 有序列表
	1. 列表1
	2. 列表2
	3. 列表3
	4. 列表4


**无序列表 : '*' 或 '-'**
* 无序列表
	* 列表1
	* 列表2
	* 列表3
	* 列表4


**引用 :> 引用内容**
> 这里是引用内容区域

**内嵌代码 : `alert('Hello World');`**
```javascript
   function show(){
   	alert("this is javascript");
   }
```
**画水平线 (HR) :--------**

---

<br>

## 3.图片插入

使用Markdown将图像插入文章，你需要在Markdown编辑器输入```![title](url)```。 这时在预览面板中会自动创建一个图像上传框。你可以从电脑桌面拖放图片(.png, .gif, .jpg)到上传框, 或者点击图片上传框使用标准的图像上传方式。 如果你想通过链接插入网络上已经存在的图片，只要单击图片上传框的左下角的“链接”图标，这时就会呈现图像URL的输入框。想给图片添加一个标题, 你需要做的是将标题文本插图中的方括号，e.g;![This is a title](http://pic.58pic.com/58pic/14/27/45/71r58PICmDM_1024.jpg)

<br>

## 4.脚注标注
使用这样的占位符号可以将脚注添加到文本中[^1]. 另外，你可以使用“n”而不是数字的[^n]所以你可以不必担心使用哪个号码[^num]。在您的文章的结尾，你可以如下图所示定义匹配的注脚，URL将变成链接:
<br>

## 5.表格显示

| 居左对齐 |  居中对齐  |  居右对齐 |
| :-----  | :------: | ------:  |
| left    | center   | right    |
| a       |   b      |     c    |
| high    |   I      |   who    |


[^1]: 脚注使用方法一  http://www.baidu.com
[^num]: [电话号码](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&rsv_idx=2&tn=baiduhome_pg&wd=%E7%94%B5%E8%AF%9D%E5%8F%B7%E7%A0%81&rsv_spt=1&oq=jekyll%2520%25E8%25A7%25A3%25E6%259E%2590%25E6%25A0%2587%25E9%25A2%2598%25E6%259C%2589%25E9%2597%25AE%25E9%25A2%2598&rsv_pq=d01410a0000bae1e&rsv_t=0ab2wQCEnk3wGlukl1VoedVFCdKXA9MtBW%2F04sV8BEkYo3D%2BIyDNuVSLbp%2F9607pg1rH&rqlang=cn&rsv_enter=1&rsv_sug3=14&rsv_sug1=9&rsv_sug7=100&rsv_sug2=0&inputT=2758&rsv_sug4=2757)
