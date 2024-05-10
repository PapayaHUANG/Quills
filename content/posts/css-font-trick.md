---
title: '字体的自适应'
date: 2024-05-10T11:51:42+08:00
draft: false
summary: ''
tags: [css]
categories: [studyNotes]
---

## 方法 1

### 使用到的属性

- `viewport`

### 实现效果

让字号实现自适应

### 基本概念

- 1vw = 1% viewport 宽度
- 1vh = 1% viewport 高度
- 1vmin = 1vw 或 1vh 取小值
- 1vmax = 1vw 或者 1vh 取大值

### 使用方法

```css
h1 {
	font-size: 5.9vw;
}
h2 {
	font-size: 3vh;
}
p {
	font-size: 2vmin;
}
```

## 方法 2

### 使用到的属性

- `viewport`
- `calc()`

### 实现效果

在选定最大最小视窗尺寸内，以及选定好字号的最大最小值，字体可以丝滑地改变大小。

### 具体公式

```css
body {
	font-size: calc(
		[minimum size] + ([maximum size] - [minimum size]) * ((
						100vw - [minimum viewport width]
					) / ([maximum viewport width] - [minimum viewport width]))
	);
}
```

### 事例

> 我希望字体在 320px 的屏幕上大小为 16px，在 1000px 的屏幕上大小为 22px

```css
html {
	font-size: 16px;
}
@media screen and (min-width: 320px) {
	html {
		font-size: calc(16px + 6 * ((100vw - 320px) / 680));
	}
}
@media screen and (min-width: 1000px) {
	html {
		font-size: 22px;
	}
}
```

## 方法 2.5

### 使用属性

- `min()`
- `max()`

### 使用效果

简化方法二的公式

### 具体操作

1. 把公式中所有`calc()`的表达都转换为`min()`和 `max()`

```css
html {
	font-size: min(max(16px, calc(16px + 6 * ((100vw - 320px) / 680))), 22px);
}
```

2. 进一步简化

```css
html {
	font-size: min(max(1rem, 4vw)), 22px);
}
```

## 方法 3

### 使用属性

- `clamp()`

### 具体操作

```css
h1 {
	--minFontSize: 32px;
	--maxFontSize: 200px;
	--scaler: 10vw;
	font-size: clamp(var(--minFontSize), var(--scaler), var(--maxFontSize));
}
```

## 参考资料

- [Viewport Sized Typography -- Chris Coyier](https://css-tricks.com/viewport-sized-typography/)
- [Fluid Typography -- Geoff Graham](https://css-tricks.com/snippets/css/fluid-typography/)
