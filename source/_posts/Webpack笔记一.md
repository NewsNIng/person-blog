---
layout: webpack
title: Webpack笔记一
date: 2020-03-20 19:41:56
tags: Webpack
categories: 技术
---

最近复习webpack，记录下自己以前模糊不清，或者当前醍醐灌顶的知识。所以并不是一篇普及性文章，是针对自己的笔记。或许也能解决你曾经遇到的问题。

<!-- more -->

### 前言

最近复习webpack，记录下自己以前模糊不清，或者当前醍醐灌顶的知识。所以并不是一篇普及性文章，是针对自己的笔记。或许也能解决你曾经遇到的问题。

#### 为何__webpack_require__使用下标


源代码中：

```javascript
import './hello'
```

=>

webpack解析：

```javascript
require('./hello')
```

=> 

生成的代码：

```javascript
__webpack_require__(1)
```


那么为什么表达式会变成下标呢？

这个就涉及到了模块标识符的解析了，按照标准，以下几种写法都能匹配同一个文件：

- require('./hello')
- require('./hello.js')
- require('hello')
- require('hello.js')

*(ps: 之前的《深入浅出Node.js》文章有讲过，JavaScript 中的 模块标识符 解析流程)*

所以，使用 __webpack_require__(1) 的好处在于，其接受的参数（数组 modules 中的索引值）与真实的模块实现时一一对应的，也就省略掉了模块标识符的解析过程（准确的说，是把解析过程提前到了构建期），从而可以获得更好的运行性能。



#### style-loader

基本上每个 webpack 项目都要配置该 loader，但是与传统的```<style>```标签插入到页面相比，该方法也存在着不可忽视的缺陷：**样式内容的生效时间被延后**。

如果遵循通常的页面优化建议，一般样式文件是插入在```<head>```中，```<script>```则放在```<body>```后，这样在文档的样式文件会被先下载解析，js文件内容则会被延后。

但是在通常使用的 style-loader 中，css的插入和解析是要在 js 执行期。因此如果页面可能会有一个短暂的无样式时间，对用户体验不好。

当然这个是可以避免的, extract-text-webpack-plugin 这个插件可以让css内容单独的放入一个文件中，这里不做细说。