# HTML（描述网页的标记语言/标签）

## 结构(包含)

声明：

```html
<!DOCTYPE html>     不区分大小写
```

头部元素

页面根元素：

```html
<html>              
```

元数据：

```html
<head>               </head>
```

中文编码：<meta charset="UTF-8">

文档标题：

```html
<title>
```



可见页面内容

<h1>  <h2>      <p1>    <p2>

## 语句

### 链接：

```html
<a 属性"链接"  >        </a>
```

### 图像

定义：

```html
<img src="url" alt="some_text">
```

图像标签<img>

源属性(src)    值：URL地址（图像储存地址）

Alt属性：预备可替换文本（无法载入图像时）

height			width

### 表格（呈现表格化数据）



定义：表格<table>				行<tr>             单元格<td>可放入表格

边框：<table border="数字（边框叠加）">  

表头：<th>放入表格并加深字体

标题：<caption>

单元格跨两列/行：<td colspan="列数">   rowspan

单元格边距：<cellpadding="数字（大小）">

单元格间距：<cellsapcing>

### 列表

定义列表：<dl>

定义列表项：<li>

加点（无序）列表：

```html
<ul>    标签
<li>Coffee</li>
</ul>
```

有序列表：<ol>

自定义列表：<dt>

​            描述：<dd>

### 区块（容器）

可与CSS一同设置一定范围内容的样式属性

​       <div>对较大内容块          <span>对部分文本

内联/区块元素：通常否/是以新行开始

用于布局

### 布局（美观）

### 表单

标签：<form>

------

HTML 表单用于**<u>收集用户的输入信息</u>**。通过表单元素

HTML 表单表示文档中的一个**区域**，此区域包含交互控件，将用户收集到的信息发送到 **Web 服务器**。

表单元素：文本域（textarea）<input type="text" name>

​					下拉列表（select）                      

 				   单选按钮（radio-buttons）          radio

​					复选框（checkbox）                   checkbox

​                    提交（submit）

























CSS

内联样式(网页)：style=样式属性（color、front-family）+具体改变

内部样式表（单文件）

```html
<style type="text/css">  </style>>
```

外部样式表（多页面）

```html
 <link rel="stylesheet" type="text/css" href="mystyle.css"
```

