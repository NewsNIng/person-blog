---
title: Chrome 浏览器渲染机制
date: 2019-09-27 16:31:13
tags: Chrome 浏览器
categories: 技术
---

Chrome浏览器从接收到数据后，是如何渲染页面的呢？
<!-- more -->

### 浏览器
#### 渲染引擎（内核）
Chrome blink(28+) Webkit(charome 27)
##### 浏览器渲染UI
---
1. 浏览器获取HTML文件，对文件进行解析，形成DOM Tree。
2. 与此同时，对CSS解析，生成Style Rules。
3. 接着将DOM Tree 与 Style Rules 合成喂 Render Tree。
4. 进入布局（Layout）阶段，为每个节点分配一个应该出现在屏幕上的确切坐标。
5. 调用GPU进行绘制（Paint），遍历Render Tree节点，将元素呈现出来。

![1570696431299.jpg](img/Chrome-浏览器渲染机制/1570696431299.jpg)

##### 浏览器解析CSS选择器
---
浏览器【**从右往左**】解析CSS选择器。  

DOM Tree与Style Rules 合成为 Render Tree，实际是将Style Rules附着至DOM Tree上，因此需要根据浏览器提供的信息对DOM Tree进行遍历，才能将样式附着到元素。  

以下css为例：
```css
.mod-nav h3 span { font-size: 16px; }
```
对应DOM Tree如下：


![1570696462005.jpg](img/Chrome-浏览器渲染机制/1570696462005.jpg)

如果按照从左往右匹配，是采用树的先序遍历，性能都浪费在失败的查找当中。
所以，浏览器采用从右往左的匹配：
1. 先找到所有最右节点span，对于每个span，向上查找h3
2. 由h3再向上查找class=mod-nav的节点。
3. 最后找到根元素html则结束这个分支的遍历。

从右往左遍历的性能更好，匹配在第一部就筛选掉了大量不符合条件的最右节点（叶子节点）。

##### DOM Tree构建
---
1. 转码String： 浏览器将接受到的二进制数据按照制定编码格式转化为HTML字符串。
2. 生成Tokens： 对HTML字符串parser解析，生成Tokens。
3. 构建Nodes：对Node节点添加特定属性，确定Node节点的父、子、兄弟等关系和所属TreeScope。
4. 生成DOM Tree：通过Node包含的指针确定的关系构建出DOM Tree。

![1570696471927.jpg](img/Chrome-浏览器渲染机制/1570696471927.jpg)

##### 重排和重绘
---
任何改变用来构建渲染树的信息都会导致一次重排或重绘：
- 添加、删除、更新DOM节点。
- 通过display: none隐藏一个DOM节点 - 触发重排和重绘。
- 通过visibility: hidden隐藏一个DOM节点 - 只会触发重绘，因为没有几何变化。
- 移动或者页面中DOM节点添加动画。
- 添加一个样式表，调整样式属性。
- 用户行为，例如调整窗口大小、改变字号、滚动等。

##### 如何避免重排或者重绘
---
**集中改变样式**
我们往往通过改变class的方式集中改变样式
```javascript
const theme = isDark ? 'dark': 'light';
ele.setAttribute('className', theme);
```
**使用DocumentFragment**
可以通过createDocumentFragment创建一个在DOM树之外的节点，然后在此节点中批量操作，最后插入到DOM树之中，因此只触发一次重排。（可以理解和离屏Canvas绘制差不多）：
```javascript
var fragment = document.createDocumentFragment();
for (let i = 0;i<10;i++){
  let node = document.createElement("p");
  node.innerHTML = i;
  fragment.appendChild(node);
}
document.body.appendChild(fragment);
```
**提升为合成层**
将元素提升为合成层有以下优点：
- 合成层的位图会交给GPU合成，比CPU处理要快。
- 当需要repaint时，只需要repaint本身，不会影响到其他层。
- 对于transform和opacity效果，不会触发layout和paint。
提升合成层最好的方式是使用CSS的will-change属性：
```css
#target {
    will-change: transform;
}
```
#### JavaScript引擎
Chrome V8
待续...