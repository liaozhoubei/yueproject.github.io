---
layout: post
title:  "自定义View绘制准备-DecorView创建"
date:   2024-04-01 01:51:34 +0800
categories: 自定义View
tag: [Android]
---

![](assets/img/docs/944365-10ea33e4c4252db5.png)

* * *

前言
==

回忆前文：[Android自定义View基础：ViewRoot、DecorView & Window的简介](https://www.jianshu.com/p/28d396a0f05f)，可看出最后1步 = 绘制  

![](assets/img/docs/944365-34992eb46bdf93e7.png)

示意图

*   但在绘制前，系统会有一些绘制准备，即前面几个步骤：创建`PhoneWindow`类、`DecorView`类、`ViewRootmpl`类等
*   今天，我将主要讲解View绘制前的准备，**主要包括：`DecorView`创建 & 显示**，希望你们会喜欢。

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

1\. DecorView的创建
================

DecorView是显示的顶层View，那么View的绘制准备从DecorView创建开始说起。

#### 源码分析

DecorView的创建开始是从大家最熟悉的setContentView()开始。

```csharp
/**
  * 具体使用：Activity的setContentView()
  */
  @Override
    public void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    }

/**
  * 源码分析：Activity的setContentView()
  */
   public void setContentView(int layoutResID) {
        // getWindow() 作用：获得Activity 的成员变量mWindow ->>分析1
        // Window类实例的setContentView（） ->>分析2
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
   }

/**
  * 分析1：成员变量mWindow
  */
  // 1. 创建一个Window对象（即 PhoneWindow实例）
  // Window类 = 抽象类，其唯一实现类 = PhoneWindow
  mWindow = new PhoneWindow(this, window);
  
  // 2. 设置回调，向Activity分发点击或状态改变等事件
  mWindow.setWindowControllerCallback(this);
  mWindow.setCallback(this);

  // 3. 为Window实例对象设置WindowManager对象
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);

    }

/**
  * 分析2：Window类实例的setContentView（）
  */
  public void setContentView(int layoutResID) {

        // 1. 若mContentParent为空，创建一个DecroView
        // mContentParent即为内容栏（content）对应的DecorView = FrameLayout子类
        if (mContentParent == null) {
            installDecor(); // ->>分析3
        } else {
            // 若不为空，则删除其中的View
            mContentParent.removeAllViews();
        }

        // 2. 为mContentParent添加子View
        // 即Activity中设置的布局文件
        mLayoutInflater.inflate(layoutResID, mContentParent);

        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();//回调通知，内容改变
        }
    }

/**
  * 分析3：installDecor()
  * 作用：创建一个DecroView
  */
  private void installDecor() {

    if (mDecor == null) {
        // 1. 生成DecorView ->>分析4
        mDecor = generateDecor(); 
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
        if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
            mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
        }
    }
    // 2. 为DecorView设置布局格式 & 返回mContentParent ->>分析5
    if (mContentParent == null) {
        mContentParent = generateLayout(mDecor); 
        ...
        } 
    }
}

/**
  * 分析4：generateDecor()
  * 作用：生成DecorView
  */
  protected DecorView generateDecor() {
        return new DecorView(getContext(), -1);
    }
    // 回到分析原处

/**
  * 分析5：generateLayout(mDecor)
  * 作用：为DecorView设置布局格式
  */
  protected ViewGroup generateLayout(DecorView decor) {

        // 1. 从主题文件中获取样式信息
        TypedArray a = getWindowStyle();

        // 2. 根据主题样式，加载窗口布局
        int layoutResource;
        int features = getLocalFeatures();

        // 3. 加载layoutResource
        View in = mLayoutInflater.inflate(layoutResource, null);

        // 4. 往DecorView中添加子View
        // 即文章开头介绍DecorView时提到的布局格式，那只是一个例子，根据主题样式不同，加载不同的布局。
        decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT)); 
        mContentRoot = (ViewGroup) in;

        // 5. 这里获取的是mContentParent = 即为内容栏（content）对应的DecorView = FrameLayout子类
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT); 
      
        return contentParent;
    }
```

#### 源码总结

1.  创建Window抽象类的子类PhoneWindow类的实例对象;
2.  为PhoneWindow类对象设置WindowManager对象;
3.  为PhoneWindow类对象创建1个DecroView类对象(根据所选的主题样式增加);
4.  为DecroView类对象中的content增加Activity中设置的布局文件。

此时，DecorView(即顶层View)已创建和添加Activity中设置的布局文件中，但目前仍未显示出来，即不可见。

* * *

2\. DecorView的显示
================

#### 源码分析

在主线程创建时，会调用handleResumeActivity()，DecorView的显示操作从此处开始。

```java
/**
  * 源码分析：主线程创建时，调用的handleResumeActivity()
  */
  @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

/**
  * 源码分析：Activity的setContentView()
  */
 final void handleResumeActivity(IBinder token,
    boolean clearHide, boolean isForward, boolean reallyResume) {

    ActivityClientRecord r = performResumeActivity(token, clearHide);

    if (r != null) {
        final Activity a = r.activity;
          if (r.window == null && !a.mFinished && willBeVisible) {
        // 1. 获取Window实例中的Decor对象
        r.window = r.activity.getWindow();
        View decor = r.window.getDecorView();

        // 2. DecorView对用户不可见
        decor.setVisibility(View.INVISIBLE);
        ViewManager wm = a.getWindowManager();
        WindowManager.LayoutParams l = r.window.getAttributes();
        a.mDecor = decor;
      
        l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;

        // 3. DecorView被添加进WindowManager了
        // 此时，还是不可见
        if (a.mVisibleFromClient) {
            a.mWindowAdded = true;
            
            wm.addView(decor, l);
        }

        // 4. 此处设置DecorView对用户可见
        if (!r.activity.mFinished && willBeVisible
            && r.activity.mDecor != null && !r.hideForNow) {
             if (r.activity.mVisibleFromClient) {
                    r.activity.makeVisible();
                    // —>>分析1
                }
            }
    }
/**
  * 分析1：Activity.makeVisible()
  */
  void makeVisible() {
   if (!mWindowAdded) {
            ViewManager wm = getWindowManager();
            // 1. 将DecorView添加到WindowManager ->>分析2
            wm.addView(mDecor, getWindow().getAttributes());
            mWindowAdded = true;
        }
        // 2. DecorView可见
        mDecor.setVisibility(View.VISIBLE);
    }

/**
  * 分析2：wm.addView
  * 作用：WindowManager = 1个接口，由WindowManagerImpl类实现
  */
  public final class WindowManagerImpl implements WindowManager {    
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();

    @Override
    public void addView(View view, ViewGroup.LayoutParams params) {
        mGlobal.addView(view, params, mDisplay, mParentWindow); ->>分析3
    }
}

/**
  * 分析3：WindowManagerGlobal 的addView()
  */
  public void addView(View view, ViewGroup.LayoutParams params,Display display, Window parentWindow) {

     final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;

     ...

     synchronized (mLock) {

     // 1. 实例化一个ViewRootImpl对象
     ViewRootImpl root;
     root = new ViewRootImpl(view.getContext(), display);
     view.setLayoutParams(wparams);

     mViews.add(view);
     mRoots.add(root);
     mParams.add(wparams);
     }

     // 2. WindowManager将DecorView实例对象交给ViewRootImpl 绘制View
     root.setView(view, wparams, panelParentView);
     // ->> 分析4
       }
    }
 }

/**
  * 分析4：ViewRootImpl.setView（）
  */
    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
                requestLayout(); // ->>分析5
    }

/**
  * 分析5：ViewRootImpl.requestLayout()
  */
    @Override
    public void requestLayout() {

        if (!mHandlingLayoutInLayoutRequest) {
            // 1. 检查是否在主线程
            checkThread();
            mLayoutRequested = true;//mLayoutRequested 是否measure和layout布局。
            // 2. ->>分析6
            scheduleTraversals();
        }
    }

/**
  * 分析6：ViewRootImpl.scheduleTraversals()
  */
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();

            // 通过mHandler.post（）发送一个runnable，在run()方法中去处理绘制流程
            // 与ActivityThread的Handler消息传递机制相似
            // ->>分析7
            mChoreographer.postCallback(Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        }
    }

/**
  * 分析7：Runnable类的子类对象mTraversalRunnable
  * 作用：在run()方法中去处理绘制流程
  */
    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal(); // ->>分析8
        }
    }
    final TraversalRunnable mTraversalRunnable = new TraversalRunnable();

/**
  * 分析8：doTraversal()
  */
    void doTraversal() {
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            performTraversals(); 
            // 最终会调用performTraversals()，从而开始View绘制的3大流程：Measure、Layout、Draw
    }

// 注：
// a. ViewRootImpl中W类是Binder的Native端，用于接收WmS处理操作
// b. 因W类的接收方法是在线程池中的，故可通过Handler将事件处理切换到主线程中
```

#### 源码总结

1.  将DecorView对象添加到WindowManager中;
2.  创建ViewRootImpl对象;
3.  WindowManager将DecorView对象交给ViewRootImpl对象;
4.  ViewRootImpl对象通过Handler向主线程发送了一条触发遍历操作的消息：performTraversals()；该方法用于执行View的绘制流程（measure、layout、draw）。

ViewRootImpl对象中接收的各种变化（如来自WmS的窗口属性变化、来自控件树的尺寸变化、重绘请求等都引发performTraversals()的调用及完成相关处理，并最终显示到可见的Activity中。整个流程如图下所示。

![](assets/img/docs/944365-40e40c28039197b3.png)

从上面的结论可以看出：

*   一次次performTraversals()的调用驱动着控件树有条不紊的工作;
*   一旦此方法无法正常执行，整个控件树都将处于僵死状态;
*   因此performTraversals()可以说是ViewRootImpl类对象的核心逻辑。而performTraversals()的后续逻辑，则是View绘制的三大流程：测量流程(measure)、布局流程(layout)、绘制流程(draw)。

* * *

3\. 总结
======

*   本文全面总结自定义`View` 绘制前的准备，主要包括：`DecorView`创建 & 显示，具体总结如下：
*   工作流程机制

![](assets/img/docs/944365-40e40c28039197b3.png)      

*   源码分析

![](assets/img/docs/944365-7628f5c6bdc57a0c.png)      
    
    示意图
    
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

本文转自 <https://www.jianshu.com/p/ac3262d233af>，如有侵权，请联系删除。