---
layout: post
title:  "ViewRoot、DecorView、Window区别是什么"
date:   2024-04-01 01:51:34 +0800
categories: 自定义View
tag: [Android]
---

* * *

前言
==

*   自定义View原理是Android开发者必须了解的基础，在了解自定义View之前，你需要有一定的知识储备。
*   今天，本文将全面解析关于自定义View中基础：**ViewRoot、DecorView & Window**，希望你们会喜欢。

> Carson带你学Android自定义View文章系列：  
> [Carson带你学Android：自定义View基础]({% post_url 2024-04-01-自定义View基础必知必会 %})   
> [Carson带你学Android：自定义View-ViewRoot、DecorView、Window区别是什么]({% post_url 2024-04-01-ViewRoot-DecorView-Window区别是什么 %})  
> [Carson带你学Android：一文梳理自定义View工作流程]({% post_url 2024-04-01-一文梳理自定义View工作流程 %})  
> [Carson带你学Android：自定义View绘制准备-DecorView创建]({% post_url 2024-04-01-自定义View绘制准备-DecorView创建 %})  
> [Carson带你学Android：自定义View Measure过程]({% post_url 2024-04-01-自定义View测量过程(Measure) %})  
> [Carson带你学Android：带你了解神秘的MeasureSpec类]({% post_url 2024-04-01-自定义View-带你了解神秘的MeasureSpec类 %})  
> [Carson带你学Android：自定义View Layout过程]({% post_url 2024-04-01-自定义View布局过程(Layout) %})  
> [Carson带你学Android：自定义View Draw过程]({% post_url 2024-04-01-自定义View绘制过程(Draw) %})  
> [Carson带你学Android：手把手教你写一个完整的自定义View]({% post_url 2024-04-01-手把手教你写一个完整的自定义View %})  
> [Carson带你学Android：Canvas类全面解析]({% post_url 2024-04-01-自定义View-Canvas类全面解析 %})  
> [Carson带你学Android：Path类全面解析]({% post_url 2024-04-01-自定义View-Path类全面解析 %})  

* * *

目录
==

![](assets/img/docs/944365-d0354efb1a6ddfcf.png)

示意图

* * *

1\. ViewRoot
============

### 1.1 简介

![](assets/img/docs/944365-b46c98a1c44620c7.png)

示意图

### 1.2 特别注意

```cpp
// 在主线程中，Activity对象被创建后：
// 1. 自动将DecorView添加到Window中 & 创建ViewRootImpll对象
root = new ViewRootImpl(view.getContent(),display);

// 3. 将ViewRootImpll对象与DecorView建立关联
root.setView(view,wparams,panelParentView)
```

* * *

2\. DecorView
=============

### 2.1 定义

顶层View，即 Android 视图树的根节点；同时也是 FrameLayout 的子类

### 2.2 作用

显示 & 加载布局。View层的事件都先经过DecorView，再传递到View

### 2.3 特别说明

内含1个竖直方向的LinearLayout，分为2部分：

1.  上 = 标题栏（titlebar）
2.  下 = 内容栏（content）

![](assets/img/docs/944365-4923b6377b032256.png)

示意图

> 在`Activity`中通过 `setContentView（）`所设置的布局文件其实是被加到内容栏之中的，成为其唯一子View = id为content的`FrameLayout`中

```cpp
// 在代码中可通过content得到对应加载的布局

// 1. 得到content
ViewGroup content = (ViewGroup)findViewById(android.R.id.content);
// 2. 得到设置的View
ViewGroup rootView = (ViewGroup) content.getChildAt(0);
```

* * *

3\. Window
==========

![](assets/img/docs/944365-3d680d03d1fd9737.png)

简介

* * *

4\. Activity
============

![](assets/img/docs/944365-dd1d1de1f3eb9bb5.png)

示意图

* * *

5\. 之间关系
========

ViewRoot、DecorView、Window和Activity的关系非常重要。

### 5.1 总结

![](assets/img/docs/944365-aeeb7d69afb2cd63.png)

示意图

### 5.2 之间的关系

ViewRoot、DecorView、Window和Activity四者共同完成view的绘制。

![](assets/img/docs/944365-fc3e390fd50484c5.png)

* * *

6\. 总结
======

*   本文全面解析关于自定义View中基础：**ViewRoot、DecorView & Window**，
*   Carson带你学Android自定义View文章系列：  
    [Carson带你学Android：自定义View基础]({% post_url 2024-04-01-自定义View基础必知必会 %})   
    [Carson带你学Android：自定义View-ViewRoot、DecorView、Window区别是什么]({% post_url 2024-04-01-ViewRoot-DecorView-Window区别是什么 %})  
    [Carson带你学Android：一文梳理自定义View工作流程]({% post_url 2024-04-01-一文梳理自定义View工作流程 %})  
    [Carson带你学Android：自定义View绘制准备-DecorView创建]({% post_url 2024-04-01-自定义View绘制准备-DecorView创建 %})  
    [Carson带你学Android：自定义View Measure过程]({% post_url 2024-04-01-自定义View测量过程(Measure) %})  
    [Carson带你学Android：带你了解神秘的MeasureSpec类]({% post_url 2024-04-01-自定义View-带你了解神秘的MeasureSpec类 %})  
    [Carson带你学Android：自定义View Layout过程]({% post_url 2024-04-01-自定义View布局过程(Layout) %})  
    [Carson带你学Android：自定义View Draw过程]({% post_url 2024-04-01-自定义View绘制过程(Draw) %})  
    [Carson带你学Android：手把手教你写一个完整的自定义View]({% post_url 2024-04-01-手把手教你写一个完整的自定义View %})  
    [Carson带你学Android：Canvas类全面解析]({% post_url 2024-04-01-自定义View-Canvas类全面解析 %})  
    [Carson带你学Android：Path类全面解析]({% post_url 2024-04-01-自定义View-Path类全面解析 %})  

* * *

欢迎关注[Carson\_Ho](https://www.jianshu.com/users/383970bef0a0/latest_articles)的简书
===============================================================================

不定期分享关于**安卓开发**的干货，追求**短、平、快**，但**却不缺深度**。

* * *

请点赞！因为你的鼓励是我写作的最大动力！
====================

本文转自 <https://www.jianshu.com/p/28d396a0f05f>，如有侵权，请联系删除。