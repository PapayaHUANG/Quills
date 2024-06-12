---
title: '写一个翻转时钟之一'
date: 2024-06-12T09:40:46+08:00
draft: false
summary: '如何用前端三件套写一个翻转时钟'
tags: [html, css, javascript]
categories: [studyNotes]
---

## 前言

写一个翻转时钟应该会触及到的知识点主要有两个：

- 如何实现元素的翻转（涉及到 CSS 3D transforms 的内容）
- 用 JS 回调函数实现时钟的动态运行

那么在这篇笔记里，主要会实践如何实现元素的翻转。

## 动态翻转的基础 CSS 知识点

### Perspective

要实现 3D 效果，就需要使用 `perspective`属性。 使用这个属性需要注意两个要点：

- 这个属性的值决定的是**人的视线到元素的距离**，所以属性值越大的话，元素变形的效果就越小；
- 如果要同时把 3D 效果应用到多个元素，并且让它们的视角统一的话，`perspective` 属性一定是写在**父元素**上，而不是每一个字元素上。

除此之外，关于`perspective`还有一个延伸的属性：`perspective-origin`,它的默认值是`center`,我们可以通过修改这个属性来调整视角的起始点。

### 3D 变换函数

在 3D 变换中，引入了一个新的轴，也就是 Z 轴， Z 轴控制的是我们的视线到元素的距离。
因此就有一共 5 钟变换函数：

- `rotateX()`
- `rotateY()`
- `rotateZ()` 在 z 轴上旋转
- `translateZ()` 在 z 轴上前后移动
- `scaleZ()` 在 z 轴上拉长

### 构造一个 3D 物体

构建一个 3D 物体可以使用一个模式：

- scene
- object
- faces

scene 用来防治 3D 物品， object 就是 3D 物品本人，而 faces 就是 3D 物品的各个面。

一般会把 `perspective` 属性运用到 scene； 在 object 上设置好 `transform-style: preserve-3d` 来确保 `perspective` 的值可以应用到物体的面上；最后在 faces 层面需要把 `position` 设置为 `absolute` 并且设置 `backface-visibility:hidden` 来实现 3D 效果。

## 构建单个翻转卡牌

在实现一整个翻转时钟之前，我们可以先实现单个翻转卡片。

### 卡片的结构

我们可以把单个翻转时钟卡片拆分成两个部分：

- 平面固定部分
- 翻转部分

### 平面固定部分

假设我们需要从数字 0 翻动到数字 1，并且是从上往下翻，那么固定部分就是上 1 下 0:
![平面固定部分](https://raw.githubusercontent.com/PapayaHUANG/images/main/img/%E6%88%AA%E5%B1%8F2024-06-12%2012.44.41.png)

```html
<div class="digit" data-digit-before="0" data-digit-after="1">
	<!-- ::before -->
	<!-- ::after -->
</div>
```

这部分的 css 主要就是首先通过`.digit`来规定好时钟卡牌的大小，然后通过`position:absolute`来定位 `::before` 和 `::after` 的位置（也就是 0 和 1 的位置）。

在设置的时候有两个需要注意的点：

1. 因为这一部分在翻转的时候位于下层，所以 `z-index` 可以设置为 `0`;
2. 展示的数字是文字的一种，在排列的时候有行高，所以需要把`line-height`设置为 `0`。

那么平面固定部分的 CSS 就是：

```css
.digit {
	position: relative;
	width: 150px;
	height: 280px;
	box-shadow: 0 0 2px 1px rgba(0, 0, 0, 0.1);
	line-height: 0;
}

.digit::before,
.digit::after {
	position: absolute;
	z-index: 0;
	overflow: hidden;
	display: flex;
	justify-content: center;
	width: 100%;
	height: 50%;
}

.digit::before {
	content: attr(data-digit-before);
	bottom: 0;
	align-items: flex-start;
}

.digit::after {
	content: attr(data-digit-after);
	top: 0;
	align-items: flex-end;
}
```

### 翻转部分

翻转部分，就可以理解为一个正面为 0，反面为 1，并且是沿着底边翻转的一个卡片
![翻转正面](https://raw.githubusercontent.com/PapayaHUANG/images/main/img/%E6%88%AA%E5%B1%8F2024-06-12%2013.41.36.png)
![翻转反面](https://raw.githubusercontent.com/PapayaHUANG/images/main/img/%E6%88%AA%E5%B1%8F2024-06-12%2013.42.23.png)

遵循前文的 `scene - object - faces` 的原则，那么我们就可以把 html 结构写成这个样子：

```html
<div class="flip-clock">
	<div class="digit" data-digit-before="0" data-digit-after="1">
		<!-- ::before -->
		<div class="card" onclick="this.classList.toggle('flipped')">
			<div class="card-face card-face-front">0</div>
			<div class="card-face card-face-back">1</div>
		</div>
		<!-- ::after -->
	</div>
</div>
```

在这段代码里面 `.flip-clock` 就是我们的 `scene`, 这样设计是因为在一个翻转时钟里面除了今天写的这个单个数字卡片以外，之后还有更多数字卡片，它们的`perspective`需要统一。

依此类推，那么`.card` 就是我们的 `object`, 我们需要在这个部分首先通过`perserve-3d`把父元素的`perspective`传递过来，同时在这个元素上设置翻转的方式和翻转的起点，即：`rotateX()`和`transform-origin:bottom`；除此之外，因为翻转卡片是叠放在上文元素的上方的，所以它的`z-index:1`。

在`card-face`的层面，除了常规的位置设置以外，还需要注意的是，因为分正反面，所以我们需要`backface-visibility:hidden`。

那么完整的 css 代码如下：

```css
.card {
	position: relative;
	z-index: 1;
	width: 100%;
	height: 50%;
	transform-style: preserve-3d;
	transform-origin: bottom;
	transform: rotateX(0);

	transition: transform 0.7s ease-in-out;
}

.card.flipped {
	transform: rotateX(-180deg);
}

.card-face {
	position: absolute;
	display: flex;
	justify-content: center;
	width: 100%;
	height: 100%;
	overflow: hidden;
	backface-visibility: hidden;
}

.card-face-front {
	align-items: flex-end;
}

.card-face-back {
	align-items: flex-start;
	transform: rotateX(-180deg);
}
```

## 总结

以上就是单个翻转卡片的 html 结构及样式写法，主要的难点就在于对前后数字的拆分，以及翻转时确定好`scene-object-faces`结构。

## 参考资料

- [The CSS attr() function got nothin' on custom properties](https://css-tricks.com/css-attr-function-got-nothin-custom-properties/)
- [Intro to CSS 3D transforms](https://3dtransforms.desandro.com/card-flip)
