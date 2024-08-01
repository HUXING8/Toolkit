# MarkDown


## 符号
1. `*`实现
2. `**(body)**`来实现粗体，如：**body**；

## 字体
使用HTML的标记方法来实现字体的样式、颜色、大小的转换：
```
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

## 图标
* 添加图标：[emojipedia](https://emojipedia.org/)

## 目录索引
1. 自动生成目录：使用`[TOC]`，但不是所有的MarkDown处理器都支持自动生成目录；
2. 手动生成目录：在文本中使用了`#`来创建各标题后，再使用`[符号](#符号)`，效果为[符号](#符号)；

⬅️[返回](../ReadMe.md)