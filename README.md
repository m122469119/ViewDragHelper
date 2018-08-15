#[UI与交互] ViewDragHelper

标签（空格分隔）：  Android:UI细节与交互

---
# 常用方法
```java
/**
 *参数1 要使用DragHelper的布局
 *参数2 灵敏度，值越大越灵敏，1.0属于正常
 *参数3 回调，这里是主要阵地
 */
ViewDragHelper.create(this, 1.0f,new ViewDragHelper.Callback() {

    /**
     * 决定是否捕获此view
     * 这里自由决定
     * @param child 待捕获的子元素
     * @param pointerId
     * @return 是否捕获
     */
    @Override
    public boolean tryCaptureView(View child, int pointerId) {
     
    }

    /**
     * 水平 拖动
     * @param child 拖动的元素
     * @param left 将要去往的位置
     * @param dx 拖动了的距离
     * @return 新位置
     */
    @Override
    public int clampViewPositionHorizontal(View child, int left, int dx) {

    }

   /**
     * 垂直拖动
     * @param child 拖动的元素
     * @param top 将要去往的位置
     * @param dy 拖动了的距离
     * @return 新位置
     */
    @Override
    public int clampViewPositionVertical(View child, int top, int dy) {
        return 0;
    }

    /**
     * 返回拖拽的范围，不对拖拽进行真正的限制，仅仅决定了动画执行速度
     * 需要子view中有点击事件时添加
     */
    @Override
    public int getViewHorizontalDragRange(View child) {
        return 0;
    }
    
    /**
     * 返回拖拽的范围，不对拖拽进行真正的限制，仅仅决定了动画执行速度
     * 需要子view中有点击事件时添加
     */
    @Override
        public int getViewVerticalDragRange(View child) {
            return super.getViewVerticalDragRange(child);
    }
    
    @Override
    public void onViewCaptured(View capturedChild, int activePointerId) {
        super.onViewCaptured(capturedChild, activePointerId);
    }
    
    /**
     * 当 view 的 position发生改变时触发
     * @param changedView 拖动的view
     * @param left 新位置 X轴
     * @param top 新位置 Y轴
     * @param dx 从上次位置 到这次位置移动的距离 X轴
     * @param dy 从上次位置 到这次位置移动的距离 Y轴
     */
    @Override
    public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
        return 0;
    }


    /**
     * 拖动动作释放
     * @param releasedChild
     * @param xvel x 轴速度  每秒移动的像素值
     * @param yvel Y 轴速度 每秒移动的像素值
     */
    @Override
    public void onViewReleased(View releasedChild, float xvel, float yvel) {
        super.onViewReleased(releasedChild, xvel, yvel);
    }

    @Override
    public void onViewDragStateChanged(int state) {
        super.onViewDragStateChanged(state);
        switch (state){
            //不拖拽的状态
            case ViewDragHelper.STATE_IDLE:
                break;
            //拖拽中    
            case ViewDragHelper.STATE_DRAGGING:
                break;
            //设置中
            case ViewDragHelper.STATE_SETTLING:
                break;
        }
 
    }
});

```

# 拖拽释放相关方法
|方法名 |作用 |
|-|-|
|settleCapturedViewAt(int finalLeft, int finalTop)|以松手前的滑动速度为初速动，在Callback的onViewReleased()让捕获到的View自动滚动到指定位置|
|flingCapturedView(int minLeft, int minTop, int maxLeft, int maxTop)|以松手前的滑动速度为初速动，在Callback的onViewReleased()让捕获到的View自动滚动到指定位置|
|smoothSlideViewTo(View child, int finalLeft, int finalTop)|指定某个View自动滚动到指定的位置，初速度为0，可在任何地方调用|

#### smoothSlideViewTo() +postInvalidate() 工作原理
查看源码发现:
smoothSlideViewTo() 

调用了 computeSettleDuration()方法得到滚动的时间
```java
int xduration = computeAxisDuration(dx, xvel, mCallback.getViewHorizontalDragRange(child));
int yduration = computeAxisDuration(dy, yvel, mCallback.getViewVerticalDragRange(child));
return (int) (xduration * xweight + yduration * yweight);
```

最终调用了  
mScroller.startScroll(startLeft, startTop, dx, dy, duration);<br>
而在smoothSlideViewTo之后调用的postInvalidate();<br>
会导致ViewGroup调用dispatchDraw()方法;<br>
在dispatchDraw()方法中,会调用 drawChild(Canvas canvas, View child, long drawingTime);<br>
drawChild()又会调用computeScroll();<br>

```java
@Override
public void computeScroll() {
    if (mDragHelper.continueSettling(true)) {
        ViewCompat.postInvalidateOnAnimation(this);
    } else {
        // 动画结束
        if (mDragHelper.getViewDragState() == ViewDragHelper.STATE_IDLE) {
        }
    }
}
```
在continueSttling(boolean b)中;
通过获取到Scroller 的scrollX scrollY改变了ChildView的位置

```java
if (dx != 0) {
    ViewCompat.offsetLeftAndRight(mCapturedView, dx);
}
if (dy != 0) {
    ViewCompat.offsetTopAndBottom(mCapturedView, dy);
}
```

# 案例解析
- 范围选择器



- 商品拖拽详情






