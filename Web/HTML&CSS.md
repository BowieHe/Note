## 前端开发流程

美术实现：网页设计师根据需求设计网页

前端工程师：前端工程师将设计做成静态网页

Java程序员：后台工程师将静态网页修改为动态网页

## 网页组成部分

三部分组成：

- 内容（结构）：是我们在页面中可以看到的数据
- 表现：内容在页面上展现的形式，布局，颜色，大小等等
- 行为：页面中元素与输入设备交互的响应，一般Javascript实现

## HTML简介

Hyper Text Markup Language
通过标签来标记要显示的网页的各个部分。网页文件本身是一种文本文件，通过在文本文件中添加标记符，告诉浏览器如何显示其中的内容

## HTML书写规范

```html
<html>		表示html的开始
  <head>		head 表示html的头部信息，通常包含三部分内容，标题，css样式，js代码
  	<title></title>		标题
  </head>
  <body>		body 标签是当前页面的显示的主体内容
  </body>
</html>		表示html的结束
```

## HTML标签介绍

- 标签格式： `<标签名>封装的数据</标签名>`

  标签名大小写不敏感

- 标签有自己的属性
  1. 基本属性：bgcolor="red"-->修改简单的基本样式效果
  2. 事件属性：onclick="alart('hello')"-->设置简单事件响应后的JS
- 标签分类
  1. 单标签格式	  `<标签名/>`  --> br换行	hr水平线
  2. 双标签格式：  `<标签名>封装的数据</标签名>`

#### 标签语法

1. 标签不能交叉嵌套
2. 标签必须正确关闭
3. 属性必须有值，而且属性值必须加引号
4. 注释不能嵌套

## 常用标签

1. front字体标签

```html
<font size="7" face="宋体" color="green">我是字体标签</font>
```

2. 特殊符号

如把`<br>`做成内容显示在文本，换成`&lt br &gt`
在HTML中所有的空白字符，都会被裁掉，如果有10个，会只剩下1个。
因此文本中的空格使用 `&nbsp`

3. 标题标签

`<h1 align=“center”>  </h1>` 其中1最大，6最小
 标题属性：left， center， right

4. 超链接

a标签 `<a href="www.baidu.com" target="_blank"> 百度 </a>`
href属性设置超链接跳转地址，百度则为页面显示内容
Target : _blank新窗口中打开  _self自己窗口中打开

5. 列表标签（无序列表）

```html
<ul type="none>  #ul为无序列表，type属性设置列表项前面的符号
          <li>赵四</li>  # li是列表项
</ul>
```

6. img标签（显示一张图片）

`<img alt="找不到了" src="./1.jpg" height="120" width="120" border="1" />`
alt设置路径不存在时，用来替换显示的文本内容
src属性设置显示图片路径（地址）相对路径- > . 相对路径    ..上一级目录
                                                               绝对路径：网址
height和width设置图片高宽
border设置图片边框

7. div，span，p标签

`<div></div>`div是块标签，默认独占一行
`<span></span>`span是内联标签，长度是封装的数据长度
`<p></p>`p是段落标签，默认上下方空一行，但是如果上面也是则只是一行

7. 表格标签

```html
<table border="1" width="123"height="123" cellspacing="0">  
  表格标签 cellspacing单元格间距
  <tr>							标签表示行
    <td align="center"><b>1.1</b></td>		标签表示单元格，一个单元格一格td, <b>为加粗显示
    <th>1.2</th>		表中第一行
  </tr>
</table>
```

8. 跨行跨列表格

```html
<td colspan="2"></td>  跨列，合并同行两列
<td rowspan="2"></td>	跨行，合并同列两行
<td rowspan="2" colspan="2"></td> 合并两行两列
```

9. 了解iframe框架标签（内嵌窗口）

`<iframe src="address" height="200" name="abc"></iframe>`
和a标签组合使用，iframe添加一个name属性，在a标签中target对象改为iframe的name即可

10. 表单标签

```html
<form action="www.baidu.com" method="post">
  username<input type="text" value="abc"/> 
  一个简单的表单,value为默认值
  pwd<input type="password"> password属性为密码，不会显示
  gender<input type="radio"name="sex"checked="checked"/>男
  			<input type="radio"name="sex"/>女
  			radio为单选框，name为一组，checked为默认选择男
  爱好<input type="checkbox"/>java  多选框
  country<select>
  	<option selected="selected"> china</option>默认选中
  </select>
  <textarea rows="10" cols="30">设置行，每行显示文本
  </textarea>
  <input type="reset"value="恢复到默认"/>表单重制按钮，恢复到默认
  <input type="submit"value="提交"/> 提交按钮
  <input type="file"value=“文件上传”/> 文件上传
  <input type="hidden"/>隐藏域，表单希望提交某些数据，不需要也不希望用户参与
</form>
```

form标签是表单标签，其中action属性设置提交的服务器地址，method为提交方式（GET，POST）

表单提交没有提交到服务器的几个原因：

1. 表单没有name属性值
2. 单选，复选，下拉菜单必须要有valie属性值，不然就是on
3. 表单项，要在form标签内

GET请求特点：

1. 浏览器地址栏中：action属性值+？+请求的参数（name=value&name=value）
2. 不安全（密码会显示出来）且有长度限制

POST请求特点

1. 浏览器中只有action的属性值，文中即`www.baidu.com`
2. 安全且没有长度限制

## CSS

在HTML中写，类似
`<div style="border:2px solid red;"></div>`

或者在head中写style如下图

```html
<head>
	<style>
    div{
      border:2px solid red;
    }
    span{
      border: 1px red;
    }
  </style>
</head>
```

或者将上述style写入css文件，在html文件中
`<link rel="stylesheet" type="text/css" href="./style.scc">`

上述为标签选择器，也可以使用id选择器
需要给标签设置id--> `<div id="001">`  
css则还是在style中 原标签名改为#id

类型选择器，使用`.class`

组合选择器：选择器1，选择器2。。N选择器{}
有选择器1或者选择器2的都适用