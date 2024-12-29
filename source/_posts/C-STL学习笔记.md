---
title: C++ STL学习笔记
date: 2021-04-27 21:02:25
tags: 
- C++
- STL
categories: C++
---
最近开始刷leetcode了，发现很多题目还是有现成的数据结构和相关方法比较好，c语言啥都自己造太费劲了。
Java有些东西太啰嗦了，自己也不是特别熟练，还是C/C++用的比较顺手，于是打算学学C++的STL模板库。
写个帖子，做一些记录。

### vector
见得最多的容器，还是数vector.
vector可以看做是一个动态的数组。
* 常见用法
初始化
vector<int> a;
a.size();
a.empty();

### queue
queue<int> a;
a.pop();
a.push(x);
a.front();
a.empty();
a.size();
### stack
stack<int> a;
a.pop();
a.push(x);
a.top();
a.empty();
a.size();