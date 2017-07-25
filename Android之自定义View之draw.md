---
title: Android之自定义View之draw
tags: Android,自定义View,View
grammar_cjkRuby: true
---

## View的draw方法

官方注释所需的步骤

		/*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. draw the background, if needed     绘制背景
         *      2. If necessary, save the canvas' layers to prepare for fading  如有需要，可以先保存当前canvas的层级数据
         *      3. Draw view's content       绘制View的内容   
         *      4. Draw children     绘制View的子类
         *      5. If necessary, draw the fading edges and restore layers  如果需要，在退出此次绘制时恢复之前的canvas的层级数据
         *      6. Draw decorations (scrollbars for instance)  绘制一些装饰的效果
         */


## 流程图

![流程图][1]


  [1]: http://upload-images.jianshu.io/upload_images/1811893-ed8f9fd302253f8e.png