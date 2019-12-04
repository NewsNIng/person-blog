---
title: HashMap笔记
date: 2019-08-22 12:09:21
tags: 数据结构
categories: 技术
---

实现底层是数组，通过Hash函数散列Key。
<!-- more -->

### 前言
HashMap 实现底层是数组，通过Hash函数散列Key

![B04EE8D7-24F4-4C5E-AA64-AE4F5B020B68.png](img/HashMap笔记/B04EE8D7-24F4-4C5E-AA64-AE4F5B020B68.png)

#### 解决Hash冲突
--- 
- 利用Key的每个字符的Ascii码
- 偏移每个字符的Ascii码作相加
- Key的数据类型也作为Hash判断

例子：abc === 97 << 0 + 98 << 1 + 99 << 2 + typeof abc

#### 优化Hash实现
通过动态扩容的方式减少冲突，维持O(1)的复杂度，当table容量超过75%，触发 rehash 方法，实现扩容 
（ rehash方法复杂度为O(n) ）
