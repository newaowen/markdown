# activity display 显示过程分析

参考博客: [http://www.cnblogs.com/samchen2009/p/3367496.html][link0]

### 1. Activity准备工作, 创建ViewRootImpl

1. ViewRootImpl中的重要成员有：surface, **mChoreophaer**(单例子, 单进程只有一个), 同时通过单例**WindowManagerGlobal**创建与**WindowManagerService**的通道
2. 在addView内，**WindowManagerImpl**拿到ViewRoot并调用其setView方法，将view, layout参数交与ViewRootImpl接管.在该函数内进行初始化工作，包括创建**InputChannel**接受用户按键输入，enable图形硬件加速，向**WindowManagerService**报道，加入到其窗口管理队列中，相关源码如下，

```
	int addToDisplay(in IWindow window,  //提供给WMS的回调接口
 		in int seq, 
     in windowManager.LayoutParams attrs,  // layout参数
     in int viewVisibility, in int layerStackId,     // display ID
     out Rect outContentInsets,                      // WMS计算后返回这个View在显示屏上的位置
     out InputChannel outInputChannel);       // 用户输入通道Handle
     
```
3. WindowStat对象，是ViewRootImpl在wms server端的代表，内部维护windowToken, 
appWindowToken

4. 最后wms将调用WindowState的**attach()**, 创建SurfaceSession, 并将SurfaceSession,WindowSession和WindowSate关联
5. **ActivityThread**发送**ACTIVITY_RESUMED**告诉Activity显示开始

### 2. Choreographer和Surface创建

概念：VSYNC垂直同步信号在android系统内由*HWComposer*产生，由choreographer接受处理

**NativeDisplayEventReceiver**是注册到HWComposer的VSYNC信号的接受器，接受到信息后调的 `dispatchVsync()`，内部handler发(消息)到ActivityThread的Looper中, Looper拿到消息后回调FrameDisplayEventReceiver的*doFrame()*, 该方法将使用Choreographer驱动绘制, 
doCallbacks(INPUT) => doCallbacks(ANIMATION) => doCallbacks(TRAVERSAL)


SurfaceControl与Surface对象, 其中涉及到SurfaceFlinger中的GRAlloc管理BufferQueue (enqueBuffer, dequeueBuffer)

### 3. view Measure/Layout/Draw

ViewRootImpl: performTraversals => measureHierarchy() => performMeasure() => getRootMeasureSpec() => measure(int, int)

### 4. 硬件加速
android3.0前几乎所有ui绘制由skia(矢量绘图库）完成，后来发现性能不行，3.0后改用hwui实现，低层实现是用opengl，Cavnas基本操作大到用*hwui*重写了，但还有少部分接口，考虑到兼容性问题，还未改写

#### canvas/Renderer/DisplayList/HardwareLayer

#### TODO


[link0]: http://www.cnblogs.com/samchen2009/p/3367496.html







