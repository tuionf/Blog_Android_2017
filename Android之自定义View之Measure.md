---
title:Android之自定义View之Measure
tags: Android,自定义Vie,View
grammar_cjkRuby: true
---

**[参考][1]**

# Measure

## 名词 

### MeasureSpec

- 封装了从父对象传递给孩子的布局所需数据（你要成为我的子控件，你要在我里面占位置，你先要知道我有多少空间吧？）
- 父控件告诉子控件，我有**多大点地和我对于空间的使用策略**
- MeasureSpec由**大小和模式**组成 

	1. UNSPECIFIED 父控件还不知道子控件的大小，对子控件也没有任何约束，说你想占多少地方就占吧。（这个一般很少用到）
    2. EXACTLY 这种状态下的控件的大小是明确的。
    3. AT_MOST 父控件对子控件说，我还不知道你的大小，我给你自由，我的地方是这么大，你按你的意愿来，但最大也只能跟我一样大了，注意哦，**可能需要二次测量**，后面会讲到


## 源码分析理解

### measureChildWithMargins方法源码 

三步走：
	1. 获取 子view 的layout参数——为了获取他的宽高以及内边距
	2. 测量childView的宽和高的MeasureSpec
	3. 确定child的大小

**其中最重要的是第二步**

``` java
protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
			
			1. 获取 子view 的layout参数——为了获取他的宽高以及内边距
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

			2. 测量childView的宽和高的MeasureSpec
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

			3. 确定child的大小
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

### getChildMeasureSpec 方法分析

   public static int getChildMeasureSpec(int spec, int padding, int childDimension) {}

三个参数 

1. int spec————传入parent的 MeasureSpec
2.  int padding————第二个参数经过计算后实际得到的是**parent已被使用的宽度和child的padding和margin消耗的宽度**——在上面的调用中

2.  int childDimension——child的的大小，这个大小并不一定是child最后的大小哦，只能说**是我们希望创建的大小**

### 三个测量方法 

   //某一个子view，多宽，多高, 内部加上了viewGroup的padding值
  -  measureChild(subView, int wSpec, int hSpec); 
   //所有子view 都是 多宽，多高, 内部调用了measureChild方法
 -   measureChildren(int wSpec, int hSpec);
   //某一个子view，多宽，多高, 内部加上了viewGroup的padding值、margin值和传入的宽高wUsed、hUsed  
-    measureChildWithMargins(subView, intwSpec, int wUsed, int hSpec, int hUsed);
   
  1.  measureChildren——调用了**measureChild**方法
  2.  三个方法最终都是调用了**measure方法**
  3.  measureChildWithMargins的wUsed和hUsed两个参数代表 **已经被使用了的宽和高大小**


``` java
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        //1、获取parent的specMode 和 specSize
	   int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

		//2、size=剩余的可用大小
        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;
		//3、通过switch语句判断parent的specMode，分别处理
        switch (specMode) {
        // Parent has imposed an exact size on us
		
		//4、parent为MeasureSpec.EXACTLY时
        case MeasureSpec.EXACTLY:
		
		// 4.1、当childDimension大于0时，表示child的大小是
        //明确指出的，如layout_width= "100dp";
          // 此时child的大小= childDimension，child的测量模式= MeasureSpec.EXACTLY
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } 
			// 4.2、此时为LayoutParams.MATCH_PARENT
    //也就是    android:layoutwidth="matchparent"
      //因为parent的大小是明确的，child要匹配parent的大小
      //那么我们就直接让child=parent的大小就好，同样，child的测量模式= MeasureSpec.EXACTLY
			else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } 
			 //4.3、此时为LayoutParams.WRAP_CONTENT
    //也就是   android:layoutwidth="wrapcontent"  
    // 这个模式需要特别对待，child说我要的大小刚好够放
    //需要展示的内容就好，而此时我们并不知道child的内容
    //需要多大的地方，暂时先把parent的size给他
			else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            。。。省略。。。
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

**总结一张图**

![](http://upload-images.jianshu.io/upload_images/1811893-3f1cb2695cd1bd05.png)

## onMeasure

从两个方向去分析onMeasure方法：
1、View.onMeasure
2、布局类的，例如. FrameLayout.onMeasure

### View.onMeasure

``` java
 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	 setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(),widthMeasureSpec),getDefaultSize(getSuggestedMinimumHeight(),heightMeasureSpec));
}
```

1. 调用setMeasuredDimension设置view的大小——setMeasuredDimension(int measuredWidth, int measuredHeight)
2. 调用getDefaultSize获取View的大小——getDefaultSize(int size, int measureSpec)
3. getSuggestedMinimumWidth获取一个建议**最小值**——ggetSuggestedMinimumWidth()

**调用顺序**

![](http://onmure0d6.bkt.clouddn.com/%E8%B0%83%E7%94%A8%E9%A1%BA%E5%BA%8F.png)

**getSuggestedMinimumWidth**

``` java
	protected int getSuggestedMinimumWidth() {
        return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
    }
```
1. mMinWidth其实就是android:minWidth=""属性设置的值
2. 假设没设置有背景的情况下，就以设置minWidth值为准
3. 如果设置有背景，那么就去背景的实际宽度与minWidth中大的一个


**getDefaultSize**

``` java
public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
		
		//1、获得MeasureSpec的mode 和 Size
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
		
		// 不常用，暂时不看
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
		
		//2、 最终返回的size就是我们MeasureSpec中测量得到的size
            result = specSize;
            break;
        }
        return result;
    }
```

1. **ATMOST与EXACTLY模式下，返回的值是一样的，此时wrapcontent与match_parent是等效的**
2. 实际开发中肯定要处理这个情况，否则wrapcontent属性是等效于matchparent


**setMeasuredDimension**


``` java
protected final void setMeasuredDimension(int   measuredWidth, int measuredHeight) {
		//1、判断是否使用视觉边界布局
	  boolean optical = isLayoutModeOptical(this);
	  //2、判断view和parentView使用的视觉边界布局是否一致
	  if (optical != isLayoutModeOptical(mParent)) {
		  //不一致时要做一些边界的处理
		  Insets insets = getOpticalInsets();
		  int opticalWidth  = insets.left + insets.right;
		  int opticalHeight = insets.top  + insets.bottom;

		  measuredWidth  += optical ? opticalWidth  : -opticalWidth;
		  measuredHeight += optical ? opticalHeight : -opticalHeight;
	  }
	  //3、重点来了，经过过滤之后调用了setMeasuredDimensionRaw方法，看来应该是这个方法设置我们的view的大小
	  setMeasuredDimensionRaw(measuredWidth, measuredHeight);
	}
```

**setMeasuredDimensionRaw方法**

``` java
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
  //最终将测量好的大小存储到mMeasuredWidth和mMeasuredHeight上，所以在测量之后我们可以通过调用getMeasuredWidth获得测量的宽、getMeasuredHeight获得高
  mMeasuredWidth = measuredWidth;
  mMeasuredHeight = measuredHeight;

  mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```
1. 可以通过调用**getMeasuredWidth获得测量的宽**
2. getMeasuredHeight获得高


**小结**

![image](http://ws4.sinaimg.cn/large/534fc2d6ly1fhvec25c6ij20jt03rweg.jpg)

测量view的顺序为measure->onMeasure-> setMeasuredDimension-> setMeasuredDimensionRaw，由setMeasuredDimensionRaw最终保存测量的数据。

### 布局类的onMeasure

**onMeasure方法**

``` java
 //这里的widthMeasureSpec、heightMeasureSpec
//其实就是我们frameLayout可用的widthMeasureSpec 、
//heightMeasureSpec
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  //1、获得frameLayout下childView的个数
  int count = getChildCount();
//2、看这里的代码我们可以根据前面的Measure图来进行分析，因为只要parent
//不是EXACTLY模式，以frameLayout为例，假设frameLayout本身还不是EXACTL模式，
 // 那么表示他的大小此时还是不确定的，从表得知，此时frameLayout的大小是根据
 //childView的最大值来设置的，这样就很好理解了，也就是childView测量好后还要再
//测量一次，因为此时frameLayout的值已经可以算出来了，对于child为MATCH_PARENT
//的，child的大小也就确定了，理解了这里，后面的代码就很 容易看懂了
  final boolean measureMatchParentChildren =
          MeasureSpec.getMode(widthMeasureSpec) != MeasureSpec.EXACTLY ||
          MeasureSpec.getMode(heightMeasureSpec) != MeasureSpec.EXACTLY;
   //3、清理存储模式为MATCH_PARENT的child的队列
  mMatchParentChildren.clear();
  //4、下面三个值最终会用来设置frameLayout的大小
  int maxHeight = 0;
  int maxWidth = 0;
  int childState = 0;
  //5、开始便利frameLayout下的所有child
  for (int i = 0; i < count; i++) {
      final View child = getChildAt(i);
      //6、小发现哦，只要mMeasureAllChildren是true，就算child是GONE也会被测量哦，
      if (mMeasureAllChildren || child.getVisibility() != GONE) {
          //7、开始测量childView 
          measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);

          //8、下面代码是获取child中的width 和height的最大值，后面用来重新设置frameLayout，有需要的话
          final LayoutParams lp = (LayoutParams) child.getLayoutParams();
          maxWidth = Math.max(maxWidth,
                  child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin);
          maxHeight = Math.max(maxHeight,
                  child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin);
          childState = combineMeasuredStates(childState, child.getMeasuredState());

        //9、如果frameLayout不是EXACTLY，
          if (measureMatchParentChildren) {
              if (lp.width == LayoutParams.MATCH_PARENT ||
                      lp.height == LayoutParams.MATCH_PARENT) {
//10、存储LayoutParams.MATCH_PARENT的child，因为现在还不知道frameLayout大小，
//也就无法设置child的大小，后面需重新测量
                  mMatchParentChildren.add(child);
              }
          }
      }
  }
  
    ....
  //11、这里开始设置frameLayout的大小
  setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),resolveSizeAndState(maxHeight, heightMeasureSpec,childState << MEASURED_HEIGHT_STATE_SHIFT));



/12、frameLayout大小确认了，我们就需要对宽或高为LayoutParams.MATCH_PARENTchild重新测量，设置大小
  count = mMatchParentChildren.size();
  if (count > 1) {
      for (int i = 0; i < count; i++) {
          final View child = mMatchParentChildren.get(i);
          final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
          final int childWidthMeasureSpec;
          if (lp.width == LayoutParams.MATCH_PARENT) {
              final int width = Math.max(0, getMeasuredWidth()
                      - getPaddingLeftWithForeground() - getPaddingRightWithForeground()
                      - lp.leftMargin - lp.rightMargin);

  //13、注意这里，为child是MATCH_PARENT类型的childWidthMeasureSpec，
  //也就是大小已经测量出来了不需要再测量了
  //通过MeasureSpec.makeMeasureSpec生成相应的MeasureSpec
              childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(
                      width, MeasureSpec.EXACTLY);
          } else {

  //14、如果不是，说明此时的child的MeasureSpec是EXACTLY的，直接获取child的MeasureSpec，
              childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,
                      getPaddingLeftWithForeground() + getPaddingRightWithForeground() +
                      lp.leftMargin + lp.rightMargin,
                      lp.width);
          }

  // 这里是对高做处理，与宽类似
          final int childHeightMeasureSpec;
          if (lp.height == LayoutParams.MATCH_PARENT) {
              final int height = Math.max(0, getMeasuredHeight()
                      - getPaddingTopWithForeground() - getPaddingBottomWithForeground()
                      - lp.topMargin - lp.bottomMargin);
              childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(
                      height, MeasureSpec.EXACTLY);
          } else {
              childHeightMeasureSpec = getChildMeasureSpec(heightMeasureSpec,
                      getPaddingTopWithForeground() + getPaddingBottomWithForeground() +
                      lp.topMargin + lp.bottomMargin,
                      lp.height);
          }

  //最终，再次测量child
          child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
      }
  }
}


```

1. onMeasure方法的参数widthMeasureSpec、heightMeasureSpec其实就是我们frameLayout**可用的widthMeasureSpec 、heightMeasureSpec**
2. frameLayout本身**不是EXACTL模式**时， 那么表示他的大小此时还是不确定的，也就**无法设置child的大小，后面需重新测量**


  [1]: http://www.cherylgood.cn/articles/2017/06/01/1496288104205.html