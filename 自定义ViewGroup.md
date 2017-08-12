---
title: 自定义ViewGroup
tags: ViewGroup,Android,自定义View
grammar_cjkRuby: true
---


# 自定义ViewGroup

## 初始化 

1. 重写构造函数——三个
2. 通过this调用 
3. init来获取自定义的属性  

``` java

	public SwipeLayout(Context context) {
    this(context,null);
    }
    
    public SwipeLayout(Context context, AttributeSet attrs) {
    this(context, attrs,0);
    }
    
    public SwipeLayout(Context context, AttributeSet attrs, int defStyleAttr) {
    	super(context, attrs, defStyleAttr);
   	 	init(context, attrs, defStyleAttr);
    }
	
```

**获取自定义属性**

布局文件 attr.xml

``` xml
	<declare-styleable name="SwipeLayout">
        <attr name="leftView" format="reference"/>
        <attr name="rightView" format="reference"/>
        <attr name="contentView1" format="reference"/>
        <attr name="canRightSwipe1" format="boolean" />
        <attr name="canLeftSwipe1" format="boolean" />
        <attr name="fraction1" format="float" />
    </declare-styleable>
```

获取属性

``` java
		TypedArray typedArray = context.obtainStyledAttributes(attrs,R.styleable.SwipeLayout);
        leftViewId = typedArray.getResourceId(R.styleable.SwipeLayout_leftView,-1);
        rightViewId = typedArray.getResourceId(R.styleable.SwipeLayout_rightView,-1);
        contentViewId = typedArray.getResourceId(R.styleable.SwipeLayout_contentView1,-1);
        canLeftSwipe = typedArray.getBoolean(R.styleable.SwipeLayout_canLeftSwipe1,true);
        canRightSwipe = typedArray.getBoolean(R.styleable.SwipeLayout_canRightSwipe1,true);
        mFraction = typedArray.getFloat(R.styleable.SwipeLayout_fraction1,0.5f);
       
	    typedArray.recycle();
```


## onMeasure

思路 

1. View是match_parent 的特殊处理，其他的正常处理 
2. 正常的最终需要调用  setMeasuredDimension 方法，需要的参数 **宽高、宽高的spec以及state**
3. match_parent 的最终需要调用 measure方法，需要参数**widthMeasureSpec,  heightMeasureSpec**
4. onMeasure前面所做的事情都是来获取这些参数

``` java
setMeasuredDimension(resolveSizeAndState(maxWidth,widthMeasureSpec,childState)
                ,resolveSizeAndState(maxHeight,heightMeasureSpec,childState << MEASURED_HEIGHT_STATE_SHIFT));
				
				
view.measure(childWidthMeasureSpec,childHeightMeasureSpec);

```


步骤 

1. 获取子View的数量 

代码 

``` java
 @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //1. 获取childView的个数
        setClickable(true);
        int count = getChildCount();
        //参考frameLayout测量代码
        final boolean measureMatchParentChildren =
                MeasureSpec.getMode(widthMeasureSpec) != MeasureSpec.EXACTLY ||
                        MeasureSpec.getMode(heightMeasureSpec) != MeasureSpec.EXACTLY;
        mMatchParentChildren.clear();
        int maxHeight = 0;
        int maxWidth = 0;
        int childState = 0;
        //遍历childViews
        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                measureChildWithMargins(child,widthMeasureSpec,0,heightMeasureSpec,0);
                final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
                maxHeight = Math.max(maxHeight,child.getMeasuredHeight()+lp.leftMargin+lp.rightMargin);
                maxWidth = Math.max(maxWidth,child.getMeasuredWidth()+lp.topMargin+lp.bottomMargin);
                childState = combineMeasuredStates(childState, child.getMeasuredState());

                if (measureMatchParentChildren){
                    if ( lp.width == LayoutParams.MATCH_PARENT || lp.height == LayoutParams.MATCH_PARENT) {
                        mMatchParentChildren.add(child);
                    }
                }
            }
        }

        //宽度和高度还要考虑背景的大小
        maxHeight = Math.max(maxHeight,getSuggestedMinimumHeight());
        maxWidth = Math.max(maxWidth,getSuggestedMinimumWidth());

//        设置具体宽高
        setMeasuredDimension(resolveSizeAndState(maxWidth,widthMeasureSpec,childState)
                ,resolveSizeAndState(maxHeight,heightMeasureSpec,childState << MEASURED_HEIGHT_STATE_SHIFT));

        count = mMatchParentChildren.size();
        if (count > 1) {
            for (int i = 0; i < count; i++) {
                View view = mMatchParentChildren.get(i);
                MarginLayoutParams lp = (MarginLayoutParams) view.getLayoutParams();

                final int childWidthMeasureSpec;
                if ( lp.width == LayoutParams.MATCH_PARENT) {
                    int width = Math.max(0,getMeasuredWidth()
                            - lp.leftMargin - lp.rightMargin);
                    childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width,MeasureSpec.EXACTLY);
                }else {
                    childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,lp.leftMargin+lp.rightMargin
                            ,lp.width);
                }

                final int childHeightMeasureSpec;
                if ( lp.height == LayoutParams.MATCH_PARENT) {
                    int height = Math.max(0,getMeasuredHeight()
                            - lp.topMargin - lp.bottomMargin);
                    childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(height,MeasureSpec.EXACTLY);
                }else{
                    childHeightMeasureSpec = getChildMeasureSpec(heightMeasureSpec,lp.topMargin+lp.bottomMargin
                            ,lp.height);
                }

                view.measure(childWidthMeasureSpec,childHeightMeasureSpec);
            }
        }
    }
```


## onLayout

思路 

1. 调用layout方法，需要四个参数
``` java
view.layout(rLeft,rTop,rRight,rBottom)
```

padding——内边距 
margin——外边距

代码

``` java
 @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {

        // 1. 先找到布局中的View
        int count = getChildCount();
        int left = 0 + getPaddingLeft();
        int right = 0 + getPaddingLeft();
        int top = 0 + getPaddingTop();
        int bottom = 0 + getPaddingTop();

        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            if (mLeftView == null && child.getId() == leftViewId) {
                mLeftView = child;
                mLeftView.setClickable(true);
            }else if (mRightView == null && child.getId() == rightViewId){
                mRightView = child;
                mRightView.setClickable(true);
            }else if (mContentView == null && child.getId() == contentViewId){
                mContentView = child;
                mContentView.setClickable(true);
            }
        }

        //2. 设置放置的位置

        if (mContentView != null) {
            mContentViewLp = (MarginLayoutParams) mContentView.getLayoutParams();
            int cTop = mContentViewLp.topMargin + top;
            int cLeft = mContentViewLp.leftMargin + left;
            int cRight = cLeft + mContentView.getMeasuredWidth();

            //TODO  此处能不能使用 ??? mContentViewLp.height
            int cBottom = cTop + mContentView.getMeasuredHeight();

            mContentView.layout(cLeft, cTop, cRight, cBottom);
        }

        if (mLeftView != null) {
            MarginLayoutParams leftViewLp = (MarginLayoutParams) mLeftView.getLayoutParams();
            int cTop = leftViewLp.topMargin + top;
            int cRight = 0 - leftViewLp.rightMargin ;
            int cLeft = 0 - mLeftView.getMeasuredWidth() + leftViewLp.rightMargin + leftViewLp.leftMargin;
            int cBottom = top + mLeftView.getMeasuredHeight() + leftViewLp.bottomMargin;

            mLeftView.layout(cLeft,cTop,cRight,cBottom);
        }

        if (mRightView != null) {
            MarginLayoutParams rightViewLp = (MarginLayoutParams) mRightView.getLayoutParams();
            int rTop = rightViewLp.topMargin + top;
            int rLeft = rightViewLp.leftMargin + mContentView.getRight() + mContentViewLp.rightMargin;
            int rRight = rLeft + mRightView.getMeasuredWidth();
            int rBottom = rTop + mRightView.getMeasuredHeight();;

            mRightView.layout(rLeft,rTop,rRight,rBottom);
        }

    }
```
## 几个常用函数 

getX() 是表示Widget相对于**自身**左上角的x坐标

getRawX() 是表示相对于**屏幕**左上角的x坐标值

### 理解 getScrollX()

![image](http://wx4.sinaimg.cn/large/534fc2d6ly1fibkkgnoagj20mi0dt7bu.jpg)

1是手机屏幕，在此区域内的人眼可以看见 
2是幕布
3是内容

getScrollX 其实获取的值，就是这块幕布在窗口左边界时候的值了，而幕布上面哪个点是原点（0，0）呢？就是**初始化时内容显示的位置**。

- 将幕布往右推动的时候，幕布在窗口左边界的值就会在0的左边（-100）
- 向左推动，则其值会是在0的右边（100）

### scrollTo()和scrollBy(x,y)
scrollTo(int x, int y) 是将View中**内容**滑动到相应的位置，参考的坐标系原点为parent View的左上角。
- **参数为正的，右移** 
- 参数为负值，左移

scrollTo()指的是移动到**指定的(x,y)位置**
而scrollBy(x,y)指的是，在当前位置在移动(x,y)个位置

### 阻止父层的View截获touch事件
调用getParent().requestDisallowInterceptTouchEvent(true);方法。一旦底层View收到touch的action后调用这个方法那么父层View就不会再调用onInterceptTouchEvent了，也无法截获以后的action

 ### ViewGroup generateLayoutParams() 方法的作用
 
 父容器生成 **子view 的布局LayoutParams**;

如果一个View想要被添加到这个容器中，这个view可以调用此方法生成和容器类匹配的布局LayoutParams，

这个方法主要是用于**父容器添加子View时调用**。

用于**生成和此容器类型相匹配的布局参数类**

### startScroll方法 
第一个参数是起始移动的x坐标值，第二个是起始移动的y坐标值，第三个第四个参数都是移到某点的坐标值，而duration 当然就是执行移动的时间
### computeScroll方法 

当startScroll执行过程中即在duration时间内，computeScrollOffset  方法会一直返回false，但当动画执行完成后会返回返加true.

``` java
@Override
    public void computeScroll() {
        //判断Scroller是否执行完毕：
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            //通知View重绘-invalidate()->onDraw()->computeScroll()
            invalidate();
        }
    }
```


### onDetachedFromWindow() 

销毁View的时候调用这个方法，我们可以在里面做一些**清理**工作（做一些收尾工作）如：取消广播注册等等

 