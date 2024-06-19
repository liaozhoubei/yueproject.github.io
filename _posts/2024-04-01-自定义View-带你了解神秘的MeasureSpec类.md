---
layout: post
title:  "带你了解神秘的MeasureSpec类"
date:   2024-04-01 01:51:34 +0800
categories: 自定义View
---

![](assets/img/docs/944365-10ea33e4c4252db5.png)

* * *

前言
==

*   在了解自定义`View`三大流程的`Measure`过程前，我们需要了解一个重要基础：`MeasureSpec`
*   今天，我将全面解析 `MeasureSpec`类的相关知识，希望你们会喜欢

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

![](assets/img/docs/944365-2ffb8e66ca225469.png)

示意图

* * *

1\. 简介
======

![](assets/img/docs/944365-4b049c81b9cf5141.png)

* * *

2\. 组成
======

测量规格(MeasureSpec)是由测量模式(mode)和测量大小(size)组成，共32位(int类型)，其中：

*   测量模式(mode)：占测量规格(MeasureSpec)的高2位；
*   测量大小(size)：占测量规格(MeasureSpec)的低30位。

![](assets/img/docs/944365-95771729a6f4977b.png)

其中，测量模式(Mode)的类型有三种

![](assets/img/docs/944365-1bb2205966af597c.png)

image.png

* * *

3\. 具体使用
========

*   测量规格(MeasureSpec)的封装类是：MeasureSpec类
*   MeasureSpec类用一个变量封装了测量模式(mode)和测量大小(size)：通过使用二进制，将测量模式（mode)和测量大小(size）打包成一个int值，并提供了打包和解包的方法，这样的做法是为了减少对象内存分配和提高存取效率。具体使用如下所示：

```cpp
// 1. 获取测量模式（Mode）
int specMode = MeasureSpec.getMode(measureSpec)

// 2. 获取测量大小（Size）
int specSize = MeasureSpec.getSize(measureSpec)

// 3. 通过Mode 和 Size 生成新的SpecMode
int measureSpec=MeasureSpec.makeMeasureSpec(size, mode);
```

* * *

4\. 源码分析
========

```java
public class MeasureSpec {

  // 进位大小 = 2的30次方
  // int的大小为32位，所以进位30位 = 使用int的32和31位做标志位
  private static final int MODE_SHIFT = 30;  
    
  // 运算遮罩：0x3为16进制，10进制为3，二进制为11
  // 3向左进位30 = 11 00000000000(11后跟30个0)  
  // 作用：用1标注需要的值，0标注不要的值。因1与任何数做与运算都得任何数、0与任何数做与运算都得0
  private static final int MODE_MASK  = 0x3 << MODE_SHIFT;  

  // UNSPECIFIED的模式设置：0向左进位30 = 00后跟30个0，即00 00000000000
  // 通过高2位
  public static final int UNSPECIFIED = 0 << MODE_SHIFT;  
  
  // EXACTLY的模式设置：1向左进位30 = 01后跟30个0 ，即01 00000000000
  public static final int EXACTLY = 1 << MODE_SHIFT;  

  // AT_MOST的模式设置：2向左进位30 = 10后跟30个0，即10 00000000000
  public static final int AT_MOST = 2 << MODE_SHIFT;  

  /**
    * makeMeasureSpec（）方法
    * 作用：根据提供的size和mode得到一个详细的测量结果吗，即measureSpec
    **/ 
      public static int makeMeasureSpec(int size, int mode) {  
          return size + mode;  
      // measureSpec = size + mode；此为二进制的加法 而不是十进制
      // 设计目的：使用一个32位的二进制数，其中：32和31位代表测量模式（mode）、后30位代表测量大小（size）
      // 例如size=100(4)，mode=AT_MOST，则measureSpec=100+10000...00=10000..00100  

      }  

  /**
    * getMode（）方法
    * 作用：通过measureSpec获得测量模式（mode）
    **/    
      public static int getMode(int measureSpec) {  
       
          return (measureSpec & MODE_MASK);  
          // 即：测量模式（mode） = measureSpec & MODE_MASK;  
          // MODE_MASK = 运算遮罩 = 11 00000000000(11后跟30个0)
          //原理：保留measureSpec的高2位（即测量模式）、使用0替换后30位
          // 例如10 00..00100 & 11 00..00(11后跟30个0) = 10 00..00(AT_MOST)，这样就得到了mode的值

      }  
  /**
    * getSize方法
    * 作用：通过measureSpec获得测量大小size
    **/       
      public static int getSize(int measureSpec) {  
       
          return (measureSpec & ~MODE_MASK);  
          // size = measureSpec & ~MODE_MASK;  
         // 原理类似上面，即 将MODE_MASK取反，也就是变成了00 111111(00后跟30个1)，将32,31替换成0也就是去掉mode，保留后30位的size  
      } 
} 
```

* * *

5\. 计算逻辑
========

View的MeasureSpec值计算取决于两个因素：

*   View自身的布局参数(LayoutParams)
*   父容器的测量规格(MeasureSpec)

即View的大小是由自身布局参数(LayoutParams)和父容器的测量规格(MeasureSpec)共同决定的。

MeasureSpec值的具体计算逻辑封装在getChildMeasureSpec()里，具体计算逻辑如下源码所示。

```cpp
/**
  * 源码分析：getChildMeasureSpec（）
  * 作用：根据父视图的MeasureSpec & 布局参数LayoutParams，计算单个子View的MeasureSpec
  * 注：子view的大小由父view的MeasureSpec值 和 子view的LayoutParams属性 共同决定
  **/
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {  
    // 参数说明
    // * @param spec 父view的详细测量值(MeasureSpec) 
    // * @param padding view当前尺寸的的内边距和外边距(padding,margin) 
    // * @param childDimension 子视图的布局参数（宽/高）

    //父view的测量模式
    int specMode = MeasureSpec.getMode(spec);     

    //父view的大小
    int specSize = MeasureSpec.getSize(spec);     
  
    //通过父view计算出的子view = 父大小-边距（父要求的大小，但子view不一定用这个值）   
    int size = Math.max(0, specSize - padding);  
  
    //子view想要的实际大小和模式（需要计算）  
    int resultSize = 0;  
    int resultMode = 0;  
  
    //通过父view的MeasureSpec和子view的LayoutParams确定子view的大小  

    // 当父view的模式为EXACITY时，父view强加给子view确切的值
    //一般是父view设置为match_parent或者固定值的ViewGroup 
    switch (specMode) {  
    case MeasureSpec.EXACTLY:  
        // 当子view的LayoutParams>0，即有确切的值  
        if (childDimension >= 0) {  
            //子view大小为子自身所赋的值，模式大小为EXACTLY  
            resultSize = childDimension;  
            resultMode = MeasureSpec.EXACTLY;  

        // 当子view的LayoutParams为MATCH_PARENT时(-1)  
        } else if (childDimension == LayoutParams.MATCH_PARENT) {  
            //子view大小为父view大小，模式为EXACTLY  
            resultSize = size;  
            resultMode = MeasureSpec.EXACTLY;  

        // 当子view的LayoutParams为WRAP_CONTENT时(-2)      
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
            //子view决定自己的大小，但最大不能超过父view，模式为AT_MOST  
            resultSize = size;  
            resultMode = MeasureSpec.AT_MOST;  
        }  
        break;  
  
    // 当父view的模式为AT_MOST时，父view强加给子view一个最大的值。（一般是父view设置为wrap_content）  
    case MeasureSpec.AT_MOST:  
        // 道理同上  
        if (childDimension >= 0) {  
            resultSize = childDimension;  
            resultMode = MeasureSpec.EXACTLY;  
        } else if (childDimension == LayoutParams.MATCH_PARENT) {  
            resultSize = size;  
            resultMode = MeasureSpec.AT_MOST;  
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
            resultSize = size;  
            resultMode = MeasureSpec.AT_MOST;  
        }  
        break;  
  
    // 当父view的模式为UNSPECIFIED时，父容器不对view有任何限制，要多大给多大
    // 多见于ListView、GridView  
    case MeasureSpec.UNSPECIFIED:  
        if (childDimension >= 0) {  
            // 子view大小为子自身所赋的值  
            resultSize = childDimension;  
            resultMode = MeasureSpec.EXACTLY;  
        } else if (childDimension == LayoutParams.MATCH_PARENT) {  
            // 因为父view为UNSPECIFIED，所以MATCH_PARENT的话子类大小为0  
            resultSize = 0;  
            resultMode = MeasureSpec.UNSPECIFIED;  
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
            // 因为父view为UNSPECIFIED，所以WRAP_CONTENT的话子类大小为0  
            resultSize = 0;  
            resultMode = MeasureSpec.UNSPECIFIED;  
        }  
        break;  
    }  
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);  
}  
```

总结如下：

![](assets/img/docs/944365-7bae3e24491a97eb.png)
示意图

其中的规律总结：（以子`View`为标准，横向观察）

![](assets/img/docs/944365-c474d16d76c4ee2e.png)
示意图

> 由于`UNSPECIFIED`模式适用于系统内部多次`measure`情况，很少用到，故此处不讨论

*   注  
    区别于顶级`View`（即`DecorView`）的测量规格`MeasureSpec`计算逻辑：取决于 **自身布局参数 & 窗口尺寸**

![](assets/img/docs/944365-7b0d3542fb6e79d5.png)
示意图

* * *

6\. 总结
======

*   本文对自定义`View`绘制流程中`Measure`过程的基础`MeasureSpec`类进行了全面介绍。
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

本文转自 <https://www.jianshu.com/p/1260a98a09e9>，如有侵权，请联系删除。