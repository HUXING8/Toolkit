# MarkDown

## 符号
1. `*`实现
2. `*body*` 来实现斜体，如*body*；
3. `**body**`来实现加粗，如：**body**；
4. `***body***`来实现斜体加粗，如***body***;
5. `~~body~~`来实现划线，如~~body~~；

## 字体
使用HTML的标记方法来实现字体的样式、颜色、大小的转换：
```html
<font face="楷体">我是楷体</font>
<font color=blue>蓝色</font>
<font color=#008000>绿色</font>
<font color=Red>红色</font>
<font size=5>尺寸</font>
```
<font face="楷体">我是楷体</font> <br>
<font color=blue>蓝色</font> <br>
<font color=#008000>绿色</font> <br>
<font color=Red>红色</font> <br>
<font size=5>尺寸</font> <br>

上述设置进行组合使用，例如：`<font face="宋体" color=#FF0000 size=4>**设置为宋体红色4大小的加粗字体**</font>` <br>
<font face="宋体" color=#FF0000 size=4>**设置为宋体红色4大小的加粗字体**</font>

## 换行
1. 硬换行：在一行的末尾加上两个以上的空格，然后按回车即换行； <font color=#FF0000 size=1.5>(Github中使用硬换行不会实现换行效果)</font> 
2. 软换行：在两行文本之间加上一个空行即可实现换行；
3. 标签换行：使用HTML的换行标签`<br>`或`</br>`或`<br/>`；

## 标题
1. 使用不同个数的`#`来实现各级别的标题
2. 在标题名下添加`=====`实现一级标题，添加`-----`实现二级标题

## 分割线
* 在一行中使用三个以上的`*`或`-`或`_`来建立一个分割线。

## 图标
* 添加图标：[emojipedia](https://emojipedia.org/)

## 目录索引
1. 自动生成目录：使用`[TOC]`，但不是所有的MarkDown处理器都支持自动生成目录；
2. 手动生成目录：在文本中使用了`#`来创建各标题后，再使用`[符号](#符号)`，效果为[符号](#符号)；

## 数学公式
* 使用`$$`字符来创建LateX格式的数学表达式，例如：
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$

## 实用软件
* Typora免费版 [Typora_0.11.18](https://github.com/zogodo/typora-0.11.18)


⬅️[返回](../ReadMe.md)