---
title: 如何在Web页面中显示FPS
categories: 技术
date: 2019-10-25 19:55:19
tags: FPS
---



某次真实面试遇到的问题，当时和面试官一起讨论了实现方案，并没有实际编码，这次花时间整理并用代码实现一下。

<!-- more -->

### 前言

那是一次复试，因为该公司有前端游戏的产品，所以正巧面试官抛出了这个问题。刚听到这个问题时还有点懵比，因为之前没有遇到过该需求，但是经过短暂停顿后，立马开始在脑子里回想 **FPS** 的概念，毕竟玩过蛮多的游戏，最直观的就是CSGO，LOL这类游戏了，FPS帧数都可以在游戏画面内看到。


#### 什么是FPS

[FPS](https://baike.baidu.com/item/FPS)全称为：**每秒传输帧数(Frames Per Second)** 


> FPS是图像领域中的定义，是指画面每秒传输帧数，通俗来讲就是指动画或视频的画面数。FPS是测量用于保存、显示动态视频的信息数量。每秒钟帧数越多，所显示的动作就会越流畅。通常，要避免动作不流畅的最低是30。某些计算机视频格式，每秒只能提供15帧。 

> FPS”也可以理解为我们常说的“刷新率（单位为Hz）”，例如我们常在CS游戏里说的“FPS值”。我们在装机选购显卡和显示器的时候，都会注意到“刷新率”。一般我们设置缺省刷新率都在75Hz（即75帧/秒）以上。例如：75Hz的刷新率刷也就是指屏幕一秒内只扫描75次，即75帧/秒。而当刷新率太低时我们肉眼都能感觉到屏幕的闪烁，不连贯，对图像显示效果和视觉感观产生不好的影响。 

> 电影以每秒24张画面的速度播放，也就是一秒钟内在屏幕上连续投射出24张静止画面。有关动画播放速度的单位是fps，其中的f就是英文单词Frame（画面、帧），p就是Per（每），s就是Second（秒）。用中文表达就是多少帧每秒，或每秒多少帧。电影是24fps，通常简称为24帧。

*简单来说，一般人能接受的最低FPS约为30Hz，基本流畅等级则需要>60Hz。*


#### 浏览器中的FPS

当时脑子里想到的第一个关键API就是 [requestAnimationFrame](
https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)，因为此次问题是用代码实现，就暂时抛开Chrome开发者工具不谈。

正常而言 requestAnimationFrame 方法在一秒内会执行 60 次，讲道理，利用这点，我们可以通过时间差来计算帧数。


### 要点分析

当时的思路是写在白纸上，首先在浏览器每次 render 时记录每次间隔的时间，这个时间就作为一帧的速度，如果超过 16.67ms，表示帧数并没有到达 60 ，那可以得到以下思路：

> T0 = 第一帧时间戳
T1 = 第二帧时间戳
Tn = 第n+1帧时间戳

T0到T1帧 临帧速度 = 一秒钟 * (1 - 0) / (T1时间戳 - T0时间戳)

因为FPS是以秒作为单位，上面计算的只是T0到T1的即时速度从而推导出FPS。

### 实现

思路已经有了，借助 requestAnimationFrame 可以很快实现

```javascript
var prev_time = new Date().getTime();

var renderCount = 0;

function step() {
	var step_now = new Date().getTime();
	var diff_time = step_now - prev_time;
	renderCount += 1;
	if(diff_time >= 1000) {
		console.log(renderCount);
		renderCount = 0;
		prev_time = step_now;
	}
	
	window.requestAnimationFrame(step);
}

window.requestAnimationFrame(step);
```
这样可以从浏览器console.log中看到每秒的帧数，但是这样也太寒酸了，为了完善这个功能还远远不止。

- 插件方式封装
- 控制监听频率，比如每100ms计算一次
- 显示在页面当中

emmm......这下完善起来可用性会比较高一点

```javascript


(function(w, d, frequency){
	
	// 实现类似$.ready功能
	var readyRE = /complete|loaded|interactive/;
	if (readyRE.test(d.readyState)) {
		init();
	} else {
		d.addEventListener("DOMContentLoaded", function domLoad(){
			d.removeEventListener("DOMContentLoaded", domLoad, false);
			init();
		});
	}

	var fps_span;
	
    // 创建FPS显示元素
	function createFps() {
		var fps_div = d.createElement('div');
		fps_div.innerHTML = "FPS: <span style='color: lightgreen'>60.0</span>";
		
		fps_div.classList.add('fps_div');
		fps_div.style.backgroundColor = "rgba(33,33,33, 0.5)";
		fps_div.style.position = "fixed";
		fps_div.style.left = 0;
		fps_div.style.top = 0;
		fps_div.style.color = "#111111";
		fps_div.style.padding = "2px 4px";
		fps_div.style.zIndex = 999999;
		
		d.body.appendChild(fps_div);
	}
	
    // 更新FPS
	function updateFps(fps) {
		fps_span = fps_span || d.querySelector('.fps_div>span');
		fps_span.innerHTML = fps.toFixed(1);
	}
	
	var prev_time = new Date().getTime();
	
	var frequency_time = 0;
	var frequency_count = 0;
	
    // 步进方法
	function step(frequency) {
		var step_now = new Date().getTime();
		var diff_time = step_now - prev_time;
		
		if(frequency_time >= frequency) {
			updateFps(1000 * frequency_count / frequency_time);
			frequency_time = 0;
			frequency_count = 0;
		}else {
			frequency_time += diff_time;
			frequency_count += 1;
		}
		
		prev_time = step_now;
		
		
		w.requestAnimationFrame(step.bind(this, frequency));
	}
	
	function init() {
		createFps();
		w.requestAnimationFrame(step.bind(this, frequency));
	}
	
}(window, document, 1000));
// 1000 为频率，单位ms
```

上面的代码可以拷贝到浏览器console中直接运行，效果呢，就是我网站页面左上角的酱紫啦。

### 随便测一测

随意挑选了几个国内视频网站测试一下，只针对本人电脑以及个人意见。

配置：锐龙1700x、1050Ti、16G、Chrome 75.0.3770.100

鼠标移入移出，频繁滚动，只测最低FPS。

网站  | 最低FPS | 
-|-|-
bilibili | 34 | 
huya | 45 | 
douyu | 47 | 

可能受各种因素影响（登录状态，产品形态，功能逻辑），大家可以自己尝试一下吧，仁者见仁智者见智了。哈哈。不管怎样，bilibili,干杯!。




