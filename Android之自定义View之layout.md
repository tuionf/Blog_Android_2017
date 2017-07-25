---
title: Android之自定义View之layout
tags: Android,自定义View,View
grammar_cjkRuby: true
---

[参考][1]
# View的测量流程

流程：
1、performTraversals内部调用performLayout开始执行布局工作
2、performLayout内部会调用layout开始进行布局
3、layout中会调用setFrame确定mTop，mLeft，mRight，mBottom的值以及判断是个点的值是否发生了变化
4、最后调用onLayout方法通知下面的childView进行布局操作


# FrameLayout布局类的View（继承自Viewgroup）

1、获取父View的内边距padding的值

``` java
final int parentLeft = getPaddingLeftWithForeground();
final int parentRight = right - left - getPaddingRightWithForeground();
final int parentTop = getPaddingTopWithForeground();
 final int parentBottom = bottom - top - getPaddingBottomWithForeground();
```


2、遍历子View，处理子View的layout_gravity属性、根据View测量后的宽和高、父View的padding值、来确定子View的布局参数
3、调用child.layout方法，对子View进行布局

``` java

final int width = child.getMeasuredWidth();
final int height = child.getMeasuredHeight();

//根据父布局和子布局的 layout_gravity属性确定布局参数
child.layout(childLeft, childTop, childLeft + width, childTop + height);

```


  [1]: http://www.cherylgood.cn/articles/2017/06/02/1496365648137.html