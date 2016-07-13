---
layout: post
title: 用CAShapeLayer画一个带动画的进度条
tags:  [iOS]
categories: [iOS开发]
author: 王颖博
excerpt: "用CAShapeLayer画一个带动画的进度条"
---

- ***CAShapeLayer的用法***

这篇博文介绍了CAShapeLayer的四种用法，先放上项目gif图。

![效果图](https://raw.githubusercontent.com/wangyingbo/testCAShapeLayer/master/gif.gif)

> 一、画一个带尖角的视图；

- 经常会在项目里遇到带小尖角的一个视图。当然我们可以考虑找UI设计师帮我们切出这样一个带尖角的图片，用UIImageView去加载它。不过这样的话有很多不便利的地方，比如说想调一下背景颜色，想改变一下尖角指向的位置等等，都需要重新找我们的设计师重新做图。而程序猿们是轻易不能求人的，so还是让我自己动手实现它吧。
- 其实这个情况在之前我是做过的，之前的项目[DEMO](https://github.com/wangyingbo/testCornerView)
  效果图如图：![以前DEMO效果图](https://raw.githubusercontent.com/wangyingbo/testCornerView/master/gif.gif)
  不过在此我们使用mask属性再做一遍。

> 二、画一个胶囊状的音量控制条，可以根据音量大小实时显示；

- 画一个胶囊状的音量控制条。

> 三、画一个圆形的进度条；

- 画一个圆形的进度条。

> 四、画一个有动画效果的进度条。

- 画一个有动画效果的进度条。

