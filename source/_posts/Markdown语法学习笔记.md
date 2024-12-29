---
title: Markdown语法学习笔记
date: 2021-04-12 13:58:22
tags:
- 笔记
- Markdown
categories: 学习笔记
---
Markdown是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML（或者HTML）文档。
个人认为，Markdown可以看做xml,html的简化版，非常适合用来纯文本编辑。
好了，下面直接进入语法部分。

### 1.标题
markdown使用#来表示标题,支持六级标题
示例：
`# 一`
`## 二`
`### 三`
`#### 四`
`##### 五`
`###### 六`

显示效果：
# 一
## 二
### 三
#### 四
##### 五
###### 六

注意：#和文字之间要加1个空格
### 2.超链接
示例：
` [百度](baidu.com)`
显示效果：
[百度](baidu.com)

**此外，也可使用html风格的写法**
示例：
`<a href="baidu.com">百度 </a>`
显示效果：
<a href="baidu.com">百度 </a>
### 3.图片
#### 3.1
示例：
`![图片名称](https://www.baidu.com/img/flexible/logo/pc/result.png)`
显示效果：
![图片名称](https://www.baidu.com/img/flexible/logo/pc/result.png)

**此外，也可使用html风格的写法**
示例：
`<img src="https://www.baidu.com/img/flexible/logo/pc/result.png">`
显示效果：
<img src="https://www.baidu.com/img/flexible/logo/pc/result.png">
#### 3.2 设置图片的大小
示例：`<img src="https://www.baidu.com/img/flexible/logo/pc/result.png"      width="10%" height="10%">`
显示效果：<img src="https://www.baidu.com/img/flexible/logo/pc/result.png" width="10%" height="10%" >
#### 3.3 设置图片居中显示

### 4.代码块
#### 4.1单行代码
使用英文反顿号"` `"表示单行代码
示例：
显示效果：123

#### 4.2多行代码