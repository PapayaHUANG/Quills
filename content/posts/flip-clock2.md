---
title: '写一个翻转时钟之二'
date: 2024-06-13T22:27:34+08:00
draft: false
summary: '如何用前端三件套写一个翻转时钟'
tags: [html, css, javascript]
categories: [studyNotes]
---

## 前言

在上一篇笔记了复盘了如何实现一个卡片的翻转，那么在这篇笔记里就来记录一下，如何通过 JS 实现时钟时间的动态更新以及翻转。

## HTML 结构

因为我要写的是包含`hour`,`minute`和 `second`的时钟，所以我会有三个重复的结构：

```html
<div class="container">
	<div class="flip-clock">
		<!-- 时 -->
		<div class="digit" id="hour">
			<!-- ::before -->
			<div class="card">
				<div class="card-face card-face-front"></div>
				<div class="card-face card-face-back"></div>
			</div>
			<!-- ::after -->
		</div>
		<!-- 分 -->
		<div class="digit" id="minute">
			<!-- ::before -->
			<div class="card">
				<div class="card-face card-face-front"></div>
				<div class="card-face card-face-back"></div>
			</div>
			<!-- ::after -->
		</div>
		<!-- 秒 -->
		<div class="digit" id="second" data-digit-before="0" data-digit-after="1">
			<!-- ::before -->
			<div class="card">
				<div class="card-face card-face-front"></div>
				<div class="card-face card-face-back"></div>
			</div>
			<!-- ::after -->
		</div>
	</div>
</div>
```

在代码里可以看出来，时、分、秒的结构式完全相同的：

```html
<div class="digit" id="hour">
	<!-- ::before -->
	<div class="card">
		<div class="card-face card-face-front"></div>
		<div class="card-face card-face-back"></div>
	</div>
	<!-- ::after -->
</div>
```

时间的分类（“时”、“分”、“秒”）是由 `id=...`来区分，其他就都是共通的，由`.digit` 到 `.card` 到两个 `.card-face`。

## JS 部分

完成了翻转时钟的基本 HTML 框架之后，就来到了计时和翻转动画部分，这部分由 JavaScript 来实现。

翻转时钟的功能可以拆解为三个部分：

- 获取 DOM 元素部分
- 时钟部分
- 翻转部分

### 获取 DOM 元素

如上文所述，时钟三个`div`的格式相同，所以我们可以先构造一个叫做 `elements` 对象，然后通过遍历的方法，丰富这个`elements`对象的细节：

```js
const elements = {
	hour: { digit: document.getElementById('hour') },
	minute: { digit: document.getElementById('minute') },
	second: { digit: document.getElementById('second') },
};

for (const key in elements) {
	const el = elements[key];
	el.card = el.digit.querySelector('.card');
	el.cardFaces = el.card.querySelectorAll('.card-face');
	el.cardFaceFront = el.cardFaces[0];
	el.cardFaceBack = el.cardFaces[1];
}
```

这样 `elements` 对象就包含了所有我们需要使用的 dom 节点。

### 时钟部分

和写所有常规的时钟插件一样，我们需要使用 JS 内置函数获取到当下的时间，并且转化为两位数的字符串：

```js
function getCurrentTime() {
	const date = new Date();
	const hour = String(date.getHours()).padStart(2, '0');
	const minute = String(date.getMinutes()).padStart(2, '0');
	const second = String(date.getSeconds()).padStart(2, '0');
	return { hour, minute, second };
}
```

和常规时钟不同的地方在于我们需要在获取当下的 `hour`,`minute`,`second`以外，还需要获取这些数值的 + 1 再转换为字符串：

```js
function getNextDigit(current, unit) {
	let next = +current + 1; //此处表达一下对 TypeScript 的想念
	if (unit.className.includes('hour')) {
		next >= 24 ? (next = '00') : (next = String(next).padStart(2, '0'));
	} else {
		next >= 60 ? (next = '00') : (next = String(next).padStart(2, '0'));
	}
	return next;
}
```

有了这两个关键函数，我们就可以结合 DOM 节点来初始化时钟了，这里就不赘述了。

### 翻转部分

这个部分是难点，因为我们的翻转动画是在原本的`.card`上，添加和删除`flipped`类来实现，如果添加和删除的时间节点没有操作好，就会出现一个卡片反复翻转，上翻之后再下翻的鬼畜现象。
为了避免以上情况发生，就应用了一个事件和一个方法：

#### 一个事件

这个事件就是[transitionend](https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionend_event),他的语法是：

```js
element.addEventListener('transitionend', () => {});
```

也就是当 css transition 事件结束的时候，就会触发你写的回调函数，那放在我们的 case 里，就是确保元素 `flipped` 动画触发之后，我们来进行一系列操作，着一系列操作就是上文说的一个方法

#### 一个方法

我一开始在回调函数里写的是直接在元素上 remove 掉`flipped`，然后更新元素内部的数字信息，发现仍然无法避免上述“鬼畜”的动画发生。后来发现，这样做不能确保`flipped`真的被清理干净了，所以这里运用的方法就是克隆一个新的节点。

```js
function () {
      //卡片翻转之后需要更新卡片内部的信息
			updateElement(value, el.digit, el.cardFaceFront, el.cardFaceBack);
			// 创建克隆卡片，是为了确保克隆卡片不会立刻翻转
      //true 是深拷贝，包含 node 内的文本
			const cardClone = el.card.cloneNode(true);
			cardClone.classList.remove('flipped');
			el.digit.replaceChild(cardClone, el.card);
			//在 elements 对象中用克隆的卡片替换原始卡片，以保证正确的引用
			el.card = cardClone;
			el.cardFaces = el.card.querySelectorAll('.card-face');
			el.cardFaceFront = el.cardFaces[0];
			el.cardFaceBack = el.cardFaces[1];
		},
    // 确保只执行一次
		{ once: true }
	);
```

写好翻转一张卡片需要执行的任务之后，就可以来编写整个时装翻转的判断逻辑，这里也不赘述了。

### 最后

最后把函数拼接在一起，就完成了一个翻转时钟的运行逻辑了。

```js
function launchClock() {
	initCurrentTime();
	setInterval(flipClock, 1000);
}
```

## 总结

写翻转时钟的运行逻辑主要有两个需要注意的点：

- 在 HTML 结构重复的情况下，可以使用遍历的方法来获取 DOM 节点（这感觉有点像 React）
- 动画的移除一定要处理干净，所以动画的时间控制以及节点的处理都很重要，而在这里使用到的是`transitionend Event` 和克隆节点。
