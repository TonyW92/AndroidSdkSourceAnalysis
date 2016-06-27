**什么是RelativeLayout**
中文名叫相对布局，可以指定内部子view之间的相互关系，从而可以绘制出更精确的UI
但是相比于LinearLayout，RelativeLayout在measure的时候每次都一定会遍历2次，性能有所降低，在实际使用的时候，尽量减少布局嵌套层数

**源码分析**
RelativeLayout作为一个ViewGroup的子类，主要起到的是一个容器的作用，所以在RelativeLayout源码中没有重载onDraw的方法
又由于RelativeLayout内部的子view都是以相对关系存在的，所以只要计算出子view的坐标，就能知道每个子view该放在布局哪个位置。所以layout的过程也十分简单

```
@Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        //  The layout has actually already been performed and the positions
        //  cached.  Apply the cached values to the children.
        final int count = getChildCount();

        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                RelativeLayout.LayoutParams st =
                        (RelativeLayout.LayoutParams) child.getLayoutParams();
                child.layout(st.mLeft, st.mTop, st.mRight, st.mBottom);
            }
        }
    }
```

经过上面的简单分析，我们大致上就可以推算出，对于RelativeLayout主要的逻辑都会放在measure过程中

老规矩，把所有代码先贴出来看看，让大家有一个大致的印象

```
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mDirtyHierarchy) {
            mDirtyHierarchy = false;
            sortChildren();
        }

        int myWidth = -1;
        int myHeight = -1;

        int width = 0;
        int height = 0;

        final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        final int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        final int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        final int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        // Record our dimensions if they are known;
        if (widthMode != MeasureSpec.UNSPECIFIED) {
            myWidth = widthSize;
        }

        if (heightMode != MeasureSpec.UNSPECIFIED) {
            myHeight = heightSize;
        }

        if (widthMode == MeasureSpec.EXACTLY) {
            width = myWidth;
        }

        if (heightMode == MeasureSpec.EXACTLY) {
            height = myHeight;
        }

        mHasBaselineAlignedChild = false;

        View ignore = null;
        int gravity = mGravity & Gravity.RELATIVE_HORIZONTAL_GRAVITY_MASK;
        final boolean horizontalGravity = gravity != Gravity.START && gravity != 0;
        gravity = mGravity & Gravity.VERTICAL_GRAVITY_MASK;
        final boolean verticalGravity = gravity != Gravity.TOP && gravity != 0;

        int left = Integer.MAX_VALUE;
        int top = Integer.MAX_VALUE;
        int right = Integer.MIN_VALUE;
        int bottom = Integer.MIN_VALUE;

        boolean offsetHorizontalAxis = false;
        boolean offsetVerticalAxis = false;

        if ((horizontalGravity || verticalGravity) && mIgnoreGravity != View.NO_ID) {
            ignore = findViewById(mIgnoreGravity);
        }

        final boolean isWrapContentWidth = widthMode != MeasureSpec.EXACTLY;
        final boolean isWrapContentHeight = heightMode != MeasureSpec.EXACTLY;

        // We need to know our size for doing the correct computation of children positioning in RTL
        // mode but there is no practical way to get it instead of running the code below.
        // So, instead of running the code twice, we just set the width to a "default display width"
        // before the computation and then, as a last pass, we will update their real position with
        // an offset equals to "DEFAULT_WIDTH - width".
        final int layoutDirection = getLayoutDirection();
        if (isLayoutRtl() && myWidth == -1) {
            myWidth = DEFAULT_WIDTH;
        }

        View[] views = mSortedHorizontalChildren;
        int count = views.length;

        for (int i = 0; i < count; i++) {
            View child = views[i];
            if (child.getVisibility() != GONE) {
                LayoutParams params = (LayoutParams) child.getLayoutParams();
                int[] rules = params.getRules(layoutDirection);

                applyHorizontalSizeRules(params, myWidth, rules);
                measureChildHorizontal(child, params, myWidth, myHeight);

                if (positionChildHorizontal(child, params, myWidth, isWrapContentWidth)) {
                    offsetHorizontalAxis = true;
                }
            }
        }

        views = mSortedVerticalChildren;
        count = views.length;
        final int targetSdkVersion = getContext().getApplicationInfo().targetSdkVersion;

        for (int i = 0; i < count; i++) {
            View child = views[i];
            if (child.getVisibility() != GONE) {
                LayoutParams params = (LayoutParams) child.getLayoutParams();
                
                applyVerticalSizeRules(params, myHeight);
                measureChild(child, params, myWidth, myHeight);
                if (positionChildVertical(child, params, myHeight, isWrapContentHeight)) {
                    offsetVerticalAxis = true;
                }

                if (isWrapContentWidth) {
                    if (isLayoutRtl()) {
                        if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                            width = Math.max(width, myWidth - params.mLeft);
                        } else {
                            width = Math.max(width, myWidth - params.mLeft - params.leftMargin);
                        }
                    } else {
                        if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                            width = Math.max(width, params.mRight);
                        } else {
                            width = Math.max(width, params.mRight + params.rightMargin);
                        }
                    }
                }

                if (isWrapContentHeight) {
                    if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                        height = Math.max(height, params.mBottom);
                    } else {
                        height = Math.max(height, params.mBottom + params.bottomMargin);
                    }
                }

                if (child != ignore || verticalGravity) {
                    left = Math.min(left, params.mLeft - params.leftMargin);
                    top = Math.min(top, params.mTop - params.topMargin);
                }

                if (child != ignore || horizontalGravity) {
                    right = Math.max(right, params.mRight + params.rightMargin);
                    bottom = Math.max(bottom, params.mBottom + params.bottomMargin);
                }
            }
        }

        if (mHasBaselineAlignedChild) {
            for (int i = 0; i < count; i++) {
                View child = getChildAt(i);
                if (child.getVisibility() != GONE) {
                    LayoutParams params = (LayoutParams) child.getLayoutParams();
                    alignBaseline(child, params);

                    if (child != ignore || verticalGravity) {
                        left = Math.min(left, params.mLeft - params.leftMargin);
                        top = Math.min(top, params.mTop - params.topMargin);
                    }

                    if (child != ignore || horizontalGravity) {
                        right = Math.max(right, params.mRight + params.rightMargin);
                        bottom = Math.max(bottom, params.mBottom + params.bottomMargin);
                    }
                }
            }
        }

        if (isWrapContentWidth) {
            // Width already has left padding in it since it was calculated by looking at
            // the right of each child view
            width += mPaddingRight;

            if (mLayoutParams != null && mLayoutParams.width >= 0) {
                width = Math.max(width, mLayoutParams.width);
            }

            width = Math.max(width, getSuggestedMinimumWidth());
            width = resolveSize(width, widthMeasureSpec);

            if (offsetHorizontalAxis) {
                for (int i = 0; i < count; i++) {
                    View child = getChildAt(i);
                    if (child.getVisibility() != GONE) {
                        LayoutParams params = (LayoutParams) child.getLayoutParams();
                        final int[] rules = params.getRules(layoutDirection);
                        if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_HORIZONTAL] != 0) {
                            centerHorizontal(child, params, width);
                        } else if (rules[ALIGN_PARENT_RIGHT] != 0) {
                            final int childWidth = child.getMeasuredWidth();
                            params.mLeft = width - mPaddingRight - childWidth;
                            params.mRight = params.mLeft + childWidth;
                        }
                    }
                }
            }
        }

        if (isWrapContentHeight) {
            // Height already has top padding in it since it was calculated by looking at
            // the bottom of each child view
            height += mPaddingBottom;

            if (mLayoutParams != null && mLayoutParams.height >= 0) {
                height = Math.max(height, mLayoutParams.height);
            }

            height = Math.max(height, getSuggestedMinimumHeight());
            height = resolveSize(height, heightMeasureSpec);

            if (offsetVerticalAxis) {
                for (int i = 0; i < count; i++) {
                    View child = getChildAt(i);
                    if (child.getVisibility() != GONE) {
                        LayoutParams params = (LayoutParams) child.getLayoutParams();
                        final int[] rules = params.getRules(layoutDirection);
                        if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_VERTICAL] != 0) {
                            centerVertical(child, params, height);
                        } else if (rules[ALIGN_PARENT_BOTTOM] != 0) {
                            final int childHeight = child.getMeasuredHeight();
                            params.mTop = height - mPaddingBottom - childHeight;
                            params.mBottom = params.mTop + childHeight;
                        }
                    }
                }
            }
        }

        if (horizontalGravity || verticalGravity) {
            final Rect selfBounds = mSelfBounds;
            selfBounds.set(mPaddingLeft, mPaddingTop, width - mPaddingRight,
                    height - mPaddingBottom);

            final Rect contentBounds = mContentBounds;
            Gravity.apply(mGravity, right - left, bottom - top, selfBounds, contentBounds,
                    layoutDirection);

            final int horizontalOffset = contentBounds.left - left;
            final int verticalOffset = contentBounds.top - top;
            if (horizontalOffset != 0 || verticalOffset != 0) {
                for (int i = 0; i < count; i++) {
                    View child = getChildAt(i);
                    if (child.getVisibility() != GONE && child != ignore) {
                        LayoutParams params = (LayoutParams) child.getLayoutParams();
                        if (horizontalGravity) {
                            params.mLeft += horizontalOffset;
                            params.mRight += horizontalOffset;
                        }
                        if (verticalGravity) {
                            params.mTop += verticalOffset;
                            params.mBottom += verticalOffset;
                        }
                    }
                }
            }
        }

        if (isLayoutRtl()) {
            final int offsetWidth = myWidth - width;
            for (int i = 0; i < count; i++) {
                View child = getChildAt(i);
                if (child.getVisibility() != GONE) {
                    LayoutParams params = (LayoutParams) child.getLayoutParams();
                    params.mLeft -= offsetWidth;
                    params.mRight -= offsetWidth;
                }
            }

        }

        setMeasuredDimension(width, height);
    }
```

在RelativeLayout的measure过程中，主要可以分为6个步骤
1.先把内部子view根据纵向关系和横向关系排序
2.初始化一些变量值
3.遍历水平关系的view
4.遍历竖直关系的view
5.baseline计算
6.宽度和高度修正

在开始分析之前，我们首先要搞清楚一件事，也是我看代码时候，遇到的一个很大的困惑：什么叫纵向关系，什么叫水平关系。

水平关系换个通俗的说法，就是左右关系
LEFT_OF，RIGHT_OF，ALIGN_LEFT，ALIGN_RIGHT，ALIGN_PARENT_LEFT，ALIGN_PARENT_RIGHT
这几个属性就是水平关系，或者左右关系，同理可得纵向关系。

接下来我们开始分析下onMeasure的过程


 1. 先把内部子view根据纵向关系和横向关系排序

首先我们会根据mDirtyHierarchy的值判断是否需要把子view排序一下
```
if (mDirtyHierarchy) {
            mDirtyHierarchy = false;
            sortChildren();
        }

@Override
    public void requestLayout() {
        super.requestLayout();
        mDirtyHierarchy = true;
    }
```
mDirtyHierarchy 这个变量的值只有在requestLayout中被更新，每次调用requestLayout都会赋值为true

然后再来看sortChildren()这个方法

```
private void sortChildren() {
        final int count = getChildCount();
        if (mSortedVerticalChildren == null || mSortedVerticalChildren.length != count) {
            mSortedVerticalChildren = new View[count];
        }

        if (mSortedHorizontalChildren == null || mSortedHorizontalChildren.length != count) {
            mSortedHorizontalChildren = new View[count];
        }

        final DependencyGraph graph = mGraph;
        graph.clear();

        for (int i = 0; i < count; i++) {
            graph.add(getChildAt(i));
        }

        graph.getSortedViews(mSortedVerticalChildren, RULES_VERTICAL);
        graph.getSortedViews(mSortedHorizontalChildren, RULES_HORIZONTAL);
    }
```

这个方法也很简单主要对左右关系和上下关系的view数组进行非空判断，然后用DependencyGraph 来完成排序，DependencyGraph 顾名思义，就是关系图的意思
我们来看下DependencyGraph 的add()方法和getSortedViews()方法

```
		void add(View view) {
            final int id = view.getId();
            //有图就有节点，根据view生成一个节点
            final Node node = Node.acquire(view);
			//如果是一个有效id就把节点加到List当中
            if (id != View.NO_ID) {
                mKeyNodes.put(id, node);
            }

            mNodes.add(node);
        }


		void getSortedViews(View[] sorted, int... rules) {
			//首先找到不依赖别的view的view，作为root节点
            final ArrayDeque<Node> roots = findRoots(rules);
            int index = 0;

            Node node;
            //读取roots下一个node，直到全部遍历完
            while ((node = roots.pollLast()) != null) {
                final View view = node.view;
                final int key = view.getId();
			//把符合规则的view加到sorted里
                sorted[index++] = view;
				//根据下面findRoots()方法的分析
				//dependents里存的是依赖别人的node
				//如果A，C依赖B，那么B的的dependents里存的是A,C
                final ArrayMap<Node, DependencyGraph> dependents = node.dependents;
                final int count = dependents.size();
                //遍历所有依赖自己的node
                for (int i = 0; i < count; i++) {
                    final Node dependent = dependents.keyAt(i);
                    //dependencies存的是被依赖node的规则和node
                    //如果A依赖B和D才能确定位置，那么dependencies = B,D
                    final SparseArray<Node> dependencies = dependent.dependencies;
					
					//移除当前node和dependencies的依赖关系
					//如果dependencies只和当前的node有依赖那么，移除之后
					//dependencies node也视为rootNode
					//如果B只被A依赖，那么移除和A的关系之后，B就不被任何Node依赖
                    dependencies.remove(key);
                    if (dependencies.size() == 0) {
                        roots.add(dependent);
                    }
                }
            }

            if (index < sorted.length) {
                throw new IllegalStateException("Circular dependencies cannot exist" + " in RelativeLayout");
            }
        }


举个列子
A依赖B，B依赖C


首先存入Root的是C，因为他没有不依赖任何节点
然后B在getSortedViews（）方法中找到了依赖自己的A，并且解除了依赖关系，所以B也为rootNode，存入roots
最后A也为rootNode，所以roots里的顺序为C,B,A

```

再来分析下findRoots这个方法，该方法比较绕，需要耐心体会下

```
 /**
         * Finds the roots of the graph. A root is a node with no dependency and
         * with [0..n] dependents.
         *
         * @param rulesFilter The list of rules to consider when building the
         *        dependencies
         *
         * @return A list of node, each being a root of the graph
         */
        private ArrayDeque<Node> findRoots(int[] rulesFilter) {
            final SparseArray<Node> keyNodes = mKeyNodes;
            final ArrayList<Node> nodes = mNodes;
            final int count = nodes.size();

            // Find roots can be invoked several times, so make sure to clear
            // all dependents and dependencies before running the algorithm
            //在计算之前，先初始化node.dependents和node.dependencies
            for (int i = 0; i < count; i++) {
                final Node node = nodes.get(i);
                node.dependents.clear();
                node.dependencies.clear();
            }

            // Builds up the dependents and dependencies for each node of the graph
            //遍历所有node，node里存了当前的view，他所依赖的关系
            for (int i = 0; i < count; i++) {
                final Node node = nodes.get(i);

                final LayoutParams layoutParams = (LayoutParams) node.view.getLayoutParams();
                //取出当前view所有的依赖关系
                final int[] rules = layoutParams.mRules;
                final int rulesCount = rulesFilter.length;

                // Look only the the rules passed in parameter, this way we build only the
                // dependencies for a specific set of rules
                //遍历当前view所拥有的依赖关系
                for (int j = 0; j < rulesCount; j++) {
                //rule的值对应的就应该是该关系对应的view的id，即被依赖view的id
                    final int rule = rules[rulesFilter[j]];
                    if (rule > 0) {
                        // The node this node depends on
                        //找到被依赖的node
                        final Node dependency = keyNodes.get(rule);
                        // Skip unknowns and self dependencies
                        if (dependency == null || dependency == node) {
                            continue;
                        }
                        //这里一定要分清楚dependencies和dependency
                        //dependents里存的是依赖别人的node
                        //dependencies存的是被依赖node的规则和node
                        //举个例子 A View toLeftOf B view
                        //A依赖B
                        //dependency.dependents.put(A, this);
                        node.dependencies.put(rule, B);
                        //这里有点比较难理解
                        // Add the current node as a dependent
                        dependency.dependents.put(node, this);
                        // Add a dependency to the current node
                        node.dependencies.put(rule, dependency);
                    }
                }
            }
            
			final ArrayDeque<Node> roots = mRoots;
            roots.clear();

            // Finds all the roots in the graph: all nodes with no dependencies\
            //再次遍历所有node
            for (int i = 0; i < count; i++) {
                final Node node = nodes.get(i);
                //如果node里存的依赖关系为0，即该view不依赖任何view
                //那么该view视为rootView
                if (node.dependencies.size() == 0) roots.addLast(node);
            }

            return roots;
        }

```



==================分割线=========================

 2.初始化一些变量的值

```
int myWidth = -1;
        int myHeight = -1;

        int width = 0;
        int height = 0;

        final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        final int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        final int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        final int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        // Record our dimensions if they are known;
        //如果不是UNSPECIFIED模式
        //那就把widthSize的值赋给myWidth 
        if (widthMode != MeasureSpec.UNSPECIFIED) {
            myWidth = widthSize;
        }
        
		//如果不是UNSPECIFIED模式
        //那就把heightSize的值赋给myHeight 
        if (heightMode != MeasureSpec.UNSPECIFIED) {
            myHeight = heightSize;
        }
		//如果是精确模式，就把myWidth和myHeight的值记下来
        if (widthMode == MeasureSpec.EXACTLY) {
            width = myWidth;
        }

        if (heightMode == MeasureSpec.EXACTLY) {
            height = myHeight;
        }

        mHasBaselineAlignedChild = false;

        View ignore = null;
        //判断是否是Gravity.START和Gravity.TOP
        //目的是确定左上角的坐标
        int gravity = mGravity & Gravity.RELATIVE_HORIZONTAL_GRAVITY_MASK;
        final boolean horizontalGravity = gravity != Gravity.START && gravity != 0;
        gravity = mGravity & Gravity.VERTICAL_GRAVITY_MASK;
        final boolean verticalGravity = gravity != Gravity.TOP && gravity != 0;

        int left = Integer.MAX_VALUE;
        int top = Integer.MAX_VALUE;
        int right = Integer.MIN_VALUE;
        int bottom = Integer.MIN_VALUE;

        boolean offsetHorizontalAxis = false;
        boolean offsetVerticalAxis = false;
	//记录ignore的view
        if ((horizontalGravity || verticalGravity) && mIgnoreGravity != View.NO_ID) {
            ignore = findViewById(mIgnoreGravity);
        }
	
	//宽度和高度是否是wrap模式
        final boolean isWrapContentWidth = widthMode != MeasureSpec.EXACTLY;
        final boolean isWrapContentHeight = heightMode != MeasureSpec.EXACTLY;

        // We need to know our size for doing the correct computation of children positioning in RTL
        // mode but there is no practical way to get it instead of running the code below.
        // So, instead of running the code twice, we just set the width to a "default display width"
        // before the computation and then, as a last pass, we will update their real position with
        // an offset equals to "DEFAULT_WIDTH - width".
        //这里谷歌工程师也告诉我们了
        //在计算子分配子view的坐标时候，需要用到父view的尺寸
        //但是在完成下面这个操作之前，我们是无法拿到一个准确值
        //所以我们先用一个默认值代替
        //在计算完之后，我们再用偏移量去更新真实坐标
        //偏移量的计算公式是offset = DEFAULT_WIDTH - width
        final int layoutDirection = getLayoutDirection();
        if (isLayoutRtl() && myWidth == -1) {
            myWidth = DEFAULT_WIDTH;
        }
```

==================分割线=========================

3.遍历水平关系的view

```
View[] views = mSortedHorizontalChildren;
        int count = views.length;

        for (int i = 0; i < count; i++) {
            View child = views[i];
            if (child.getVisibility() != GONE) {
                LayoutParams params = (LayoutParams) child.getLayoutParams();
                //根据方向获得子view中设置的规则
                int[] rules = params.getRules(layoutDirection);
			//把这些左右方向的规则转化成左右坐标
                applyHorizontalSizeRules(params, myWidth, rules);
                //然后测算出水平方向的子view尺寸
                measureChildHorizontal(child, params, myWidth, myHeight);
		//确定水平方向子view位置
                if (positionChildHorizontal(child, params, myWidth, isWrapContentWidth)) {
                    offsetHorizontalAxis = true;
                }
            }
        }
```

然后我们详细的看下上面提到这些方法是如何实现的

```
private void applyHorizontalSizeRules(LayoutParams childParams, int myWidth, int[] rules) {
        RelativeLayout.LayoutParams anchorParams;

        // VALUE_NOT_SET indicates a "soft requirement" in that direction. For example:
        // left=10, right=VALUE_NOT_SET means the view must start at 10, but can go as far as it
        // wants to the right
        // left=VALUE_NOT_SET, right=10 means the view must end at 10, but can go as far as it
        // wants to the left
        // left=10, right=20 means the left and right ends are both fixed
        childParams.mLeft = VALUE_NOT_SET;
        childParams.mRight = VALUE_NOT_SET;
	
	//首先得到当前子viewde的layout_toLeftOf属性对应的view
        anchorParams = getRelatedViewParams(rules, LEFT_OF);
        if (anchorParams != null) {
        //如果设置了这个属性，那么当前子view的右坐标就是
        //layout_toLeftOf属性对应的view的左边坐标减去对应view的marginLeft的值和自身的marginRight的值
            childParams.mRight = anchorParams.mLeft - (anchorParams.leftMargin +
                    childParams.rightMargin);
	//如果alignWithParent是true
	//alignWithParent值取的是alignWithParentIfMissing
	//如果toLeftOf的view，也就是被依赖的view是null，或者gone
	//alignWithParentIfMissing这个值就会起作用
	//它会把RelativeLayout当做被依赖的对象
        } else if (childParams.alignWithParent && rules[LEFT_OF] != 0) {
		//如果父容器RelativeLayout的宽度大于0
		//那么子view的右边界就是父容器的宽度减去paddingRight和自身的marginRight
		//这么说比较抽象，可以在草稿纸上画一下，或者写个demo跑一下
            if (myWidth >= 0) {
                childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
            }
        }
	
	//同样，得到android:layout_toRightOf属性
        anchorParams = getRelatedViewParams(rules, RIGHT_OF);
        if (anchorParams != null) {
            childParams.mLeft = anchorParams.mRight + (anchorParams.rightMargin +
                    childParams.leftMargin);
        //同理
        } else if (childParams.alignWithParent && rules[RIGHT_OF] != 0) {
            childParams.mLeft = mPaddingLeft + childParams.leftMargin;
        }
//得到android:layout_alignLeft属性
        anchorParams = getRelatedViewParams(rules, ALIGN_LEFT);
        if (anchorParams != null) {
            childParams.mLeft = anchorParams.mLeft + childParams.leftMargin;
        } else if (childParams.alignWithParent && rules[ALIGN_LEFT] != 0) {
            childParams.mLeft = mPaddingLeft + childParams.leftMargin;
        }
//得到android:layout_alignRight属性
        anchorParams = getRelatedViewParams(rules, ALIGN_RIGHT);
        if (anchorParams != null) {
            childParams.mRight = anchorParams.mRight - childParams.rightMargin;
        } else if (childParams.alignWithParent && rules[ALIGN_RIGHT] != 0) {
            if (myWidth >= 0) {
                childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
            }
        }
//对android:layout_alignParentLeft进行处理，即将自己置于父容器的左边
        if (0 != rules[ALIGN_PARENT_LEFT]) {
            childParams.mLeft = mPaddingLeft + childParams.leftMargin;
        }
//将自己置于父容器的右边
        if (0 != rules[ALIGN_PARENT_RIGHT]) {
            if (myWidth >= 0) {
                childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
            }
        }
    }
```

看代码是一件枯燥的事，但是想要弄明白原理也只能看代码

我们接着看
```
private void measureChildHorizontal(View child, LayoutParams params, int myWidth, int myHeight) {
	//获得child的宽度MeasureSpec，这个方法在下面会继续分析
        int childWidthMeasureSpec = getChildMeasureSpec(params.mLeft,
                params.mRight, params.width,
                params.leftMargin, params.rightMargin,
                mPaddingLeft, mPaddingRight,
                myWidth);
        int maxHeight = myHeight;
        //mMeasureVerticalWithPaddingMargin这个变量
        //是Android用于解决一个系统bug
        //在当前系统版本号大于等于18，即大于等于4.3版本，赋值为true
        //也就是说在大于等于4.3时，测绘时，不带上margin和padding属性
        if (mMeasureVerticalWithPaddingMargin) {
            maxHeight = Math.max(0, myHeight - mPaddingTop - mPaddingBottom -
                    params.topMargin - params.bottomMargin);
        }
        int childHeightMeasureSpec;
        //mAllowBrokenMeasureSpecs在小于等于4.2时为true
        if (myHeight < 0 && !mAllowBrokenMeasureSpecs) {
            if (params.height >= 0) {
                childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(
                        params.height, MeasureSpec.EXACTLY);
            } else {
                // Negative values in a mySize/myWidth/myWidth value in RelativeLayout measurement
                // is code for, "we got an unspecified mode in the RelativeLayout's measurespec."
                // Carry it forward.
                childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
            }
        //如果子view的宽度是精确模式（MATCH或者dimens），那么它的高度
        //也是精确模式
        } else if (params.width == LayoutParams.MATCH_PARENT) {
            childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(maxHeight, MeasureSpec.EXACTLY);
        } else {
        //如果子view的宽度是最大模式，那么它的高度
        //也是最大模式
            childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(maxHeight, MeasureSpec.AT_MOST);
        }
        //拿到了子view的WidthMeasureSpec和HeightMeasureSpec
        //那么子view就能回调自己的onMeasure算出自己的尺寸
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

然后我们继续看下getChildMeasureSpec（...）这个方法

```
private int getChildMeasureSpec(int childStart, int childEnd,
            int childSize, int startMargin, int endMargin, int startPadding,
            int endPadding, int mySize) {
        int childSpecMode = 0;
        int childSpecSize = 0;

        // Negative values in a mySize value in RelativeLayout
        // measurement is code for, "we got an unspecified mode in the
        // RelativeLayout's measure spec."
        //这个跟之前measureChildHorizontal里的判断一样
        ///如果父容器RelativeLayout宽度小于，并且系统版本小于等于4.2
        if (mySize < 0 && !mAllowBrokenMeasureSpecs) {
        //如果子view的左边距和右边距都不等于VALUE_NOT_SET
        //并且右边距坐标大于左边距坐标
        //就把 childEnd - childStart当做宽度赋给子view，模式为精确模式
        //VALUE_NOT_SET值是Integer.MIN
            if (childStart != VALUE_NOT_SET && childEnd != VALUE_NOT_SET) {
                // Constraints fixed both edges, so child has an exact size.
                childSpecSize = Math.max(0, childEnd - childStart);
                childSpecMode = MeasureSpec.EXACTLY;
            } else if (childSize >= 0) {
            //如果不满足上面那个条件，但是childSize的值大于0
            //就把childSize的值赋给子view
                // The child specified an exact size.
                childSpecSize = childSize;
                childSpecMode = MeasureSpec.EXACTLY;
            } else {
            //都不满足，就把子view的模式改为UNSPECIFIED
                // Allow the child to be whatever size it wants.
                childSpecSize = 0;
                childSpecMode = MeasureSpec.UNSPECIFIED;
            }

            return MeasureSpec.makeMeasureSpec(childSpecSize, childSpecMode);
        }
	//上面的情况都基于父容器RelativeLayout小于0
	//接下来就开始判断RelativeLayout大于0的情况
        // Figure out start and end bounds.
        int tempStart = childStart;
        int tempEnd = childEnd;

        // If the view did not express a layout constraint for an edge, use
        // view's margins and our padding
        //如果没有指定start的值，就设置默认值
        //默认值就为padding+margin的和
        if (tempStart == VALUE_NOT_SET) {
            tempStart = startPadding + startMargin;
        }
        //这个就和start一个流程
        if (tempEnd == VALUE_NOT_SET) {
            tempEnd = mySize - endPadding - endMargin;
        }
	//指定最最大可以提供的大小
        // Figure out maximum size available to this view
        int maxAvailable = tempEnd - tempStart;
	//第一种情况childStart和childEnd都为有效值
	//那么子view已经提出自己需要多少大小，相当于LinearLayout
	//里的dimens情况，那么父容器就会把指定大小给子view，并且为精确模式
        if (childStart != VALUE_NOT_SET && childEnd != VALUE_NOT_SET) {
            // Constraints fixed both edges, so child must be an exact size
            childSpecMode = MeasureSpec.EXACTLY;
            childSpecSize = maxAvailable;
        } else {
        //如果左右边界（竖直模式下为上下边界）不为有效值，那么就判断
        //childSize的值
        //这里的逻辑就相当于LinearLayout里的dimens条件
            if (childSize >= 0) {
                // Child wanted an exact size. Give as much as possible
                childSpecMode = MeasureSpec.EXACTLY;

                if (maxAvailable >= 0) {
                    // We have a maxmum size in this dimension.
                    childSpecSize = Math.min(maxAvailable, childSize);
                } else {
                    // We can grow in this dimension.
                    childSpecSize = childSize;
                }
            //这部分应该很眼熟，和LinearLayout的判断是一致的
            } else if (childSize == LayoutParams.MATCH_PARENT) {
                // Child wanted to be as big as possible. Give all available
                // space
                childSpecMode = MeasureSpec.EXACTLY;
                childSpecSize = maxAvailable;
            } else if (childSize == LayoutParams.WRAP_CONTENT) {
                // Child wants to wrap content. Use AT_MOST
                // to communicate available space if we know
                // our max size
                if (maxAvailable >= 0) {
                    // We have a maximum size in this dimension.
                    childSpecMode = MeasureSpec.AT_MOST;
                    childSpecSize = maxAvailable;
                } else {
                    // We can grow in this dimension. Child can be as big as it
                    // wants
                    childSpecMode = MeasureSpec.UNSPECIFIED;
                    childSpecSize = 0;
                }
            }
        }

        return MeasureSpec.makeMeasureSpec(childSpecSize, childSpecMode);
    }
```

上面这部分代码就完成了水平方向的子view的第一次测绘，确定了大小
接下来就应该根据大小，来确定把子view放在父容器的哪个位置了

```
private boolean positionChildHorizontal(View child, LayoutParams params, int myWidth,
            boolean wrapContent) {
	//获得RelativeLayout的布局方向
	//mx.core.LayoutDirection.RTL（右到左）
	//mx.core.LayoutDirection.LTR（左到右）
        final int layoutDirection = getLayoutDirection();
        int[] rules = params.getRules(layoutDirection);
	
	//如果左边界为无效值，右边界为有效值
	//那么就根据右边界的值计算出左边界
        if (params.mLeft == VALUE_NOT_SET && params.mRight != VALUE_NOT_SET) {
            // Right is fixed, but left varies
            params.mLeft = params.mRight - child.getMeasuredWidth();
        //如果右边界为无效值，左边界为有效值
	//那么就根据左边界的值计算出右边界
        } else if (params.mLeft != VALUE_NOT_SET && params.mRight == VALUE_NOT_SET) {
            // Left is fixed, but right varies
            params.mRight = params.mLeft + child.getMeasuredWidth();
        //如果左右边界都无效
        //
        } else if (params.mLeft == VALUE_NOT_SET && params.mRight == VALUE_NOT_SET) {
            // Both left and right vary
            //如果设置了CENTER_IN_PARENT或者CENTER_HORIZONTAL
            if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_HORIZONTAL] != 0) {
            //如果不是wrap模式（一般就是match或者dimens）
            //int childWidth = child.getMeasuredWidth();
            //int left = (myWidth - childWidth) / 2;

            //params.mLeft = left;
            //params.mRight = left + childWidth;
            //这个算法就是把子view水平中心固定在RelativeLayout的中心
            //如果不明白可以在草稿纸上画下
                if (!wrapContent) {
                    centerHorizontal(child, params, myWidth);
                } else {
                //如果是wrap模式
                //左边就就等于padding+margin
                //右边距等于左边距加上测量宽度
                    params.mLeft = mPaddingLeft + params.leftMargin;
                    params.mRight = params.mLeft + child.getMeasuredWidth();
                }
                return true;
            } else {
                // This is the default case. For RTL we start from the right and for LTR we start
                // from the left. This will give LEFT/TOP for LTR and RIGHT/TOP for RTL.
                //如果是RTL（右到左）
                if (isLayoutRtl()) {
                //因为是右到左，就需要先计算右边界
                    params.mRight = myWidth - mPaddingRight- params.rightMargin;
                    params.mLeft = params.mRight - child.getMeasuredWidth();
                } else {
                //如果是左到右
                    params.mLeft = mPaddingLeft + params.leftMargin;
                    params.mRight = params.mLeft + child.getMeasuredWidth();
                }
            }
        }
        return rules[ALIGN_PARENT_END] != 0;
    }
```

然后我们再回头看下刚提到的一段代码
```

//确定水平方向子view位置
                if (positionChildHorizontal(child, params, myWidth, isWrapContentWidth)) {
                    offsetHorizontalAxis = true;
                }
                
```
positionChildHorizontal（...）的返回值有3种情况返回true
CENTER_IN_PARENT
CENTER_HORIZONTAL
ALIGN_PARENT_END

这种我们的水平测量就结束了
==================分割线=========================

4.遍历竖直关系的view

紧接着开始对有竖直方向关系的子view进行遍历

这部分只讲解下和水平方向关系不同的地方

```
for (int i = 0; i < count; i++) {
            View child = views[i];
            if (child.getVisibility() != GONE) {
                LayoutParams params = (LayoutParams) child.getLayoutParams();
                //这里和水平方向关系一样 把竖直方向的关系转化成边界
                applyVerticalSizeRules(params, myHeight);
                //这里和之前的水平关系的measureChildHorizontal有所不同
                //在下面会稍微分析一下
                measureChild(child, params, myWidth, myHeight);
                //这里参考下水平方向的分析就好了
                if (positionChildVertical(child, params, myHeight, isWrapContentHeight)) {
                    offsetVerticalAxis = true;
                }

		//从这里开始，就属于竖直方向关系遍历独有的逻辑了
		//首先判断是否是wrap模式
                if (isWrapContentWidth) {
                //然后根据RTL还是LTR和版本进行细分
                //从之前的代码看，4.2是一个分界线，在4.3之后会对margin做处理
                    if (isLayoutRtl()) {
                        if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                            width = Math.max(width, myWidth - params.mLeft);
                        } else {
                            width = Math.max(width, myWidth - params.mLeft - params.leftMargin);
                        }
                    } else {
                        if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                            width = Math.max(width, params.mRight);
                        } else {
                            width = Math.max(width, params.mRight + params.rightMargin);
                        }
                    }
                }
			//这里也是同样的道理
                if (isWrapContentHeight) {
                    if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                        height = Math.max(height, params.mBottom);
                    } else {
                        height = Math.max(height, params.mBottom + params.bottomMargin);
                    }
                }

                if (child != ignore || verticalGravity) {
                    left = Math.min(left, params.mLeft - params.leftMargin);
                    top = Math.min(top, params.mTop - params.topMargin);
                }

                if (child != ignore || horizontalGravity) {
                    right = Math.max(right, params.mRight + params.rightMargin);
                    bottom = Math.max(bottom, params.mBottom + params.bottomMargin);
                }
            }
        }
```

==================分割线=========================

5.baseline计算

这部分没有什么好看的，就是计算下baseLine
```
if (mHasBaselineAlignedChild) {
            for (int i = 0; i < count; i++) {
                View child = getChildAt(i);
                if (child.getVisibility() != GONE) {
                    LayoutParams params = (LayoutParams) child.getLayoutParams();
                    alignBaseline(child, params);

                    if (child != ignore || verticalGravity) {
                        left = Math.min(left, params.mLeft - params.leftMargin);
                        top = Math.min(top, params.mTop - params.topMargin);
                    }

                    if (child != ignore || horizontalGravity) {
                        right = Math.max(right, params.mRight + params.rightMargin);
                        bottom = Math.max(bottom, params.mBottom + params.bottomMargin);
                    }
                }
            }
        }
```

==================分割线=========================

6.宽度和高度修正

这部分代码比较好理解，在onMeasure()之前就提到
	**//这里谷歌工程师也告诉我们了
        //在计算子分配子view的坐标时候，需要用到父view的尺寸
        //但是在完成下面这个操作之前，我们是无法拿到一个准确值
        //所以我们先用一个默认值代替
        //在计算完之后，我们再用偏移量去更新真实坐标
        //偏移量的计算公式是offset = DEFAULT_WIDTH - width**
 这里就是拿到准确的宽度和高度之后，对一些依赖父容器决定位置的子view重新做一次测量

```
//又是熟悉的代码，如果是wrap模式
if (isWrapContentWidth) {
            // Width already has left padding in it since it was calculated by looking at
            // the right of each child view
            width += mPaddingRight;

            if (mLayoutParams != null && mLayoutParams.width >= 0) {
                width = Math.max(width, mLayoutParams.width);
            }

            width = Math.max(width, getSuggestedMinimumWidth());
            width = resolveSize(width, widthMeasureSpec);
		//在得到最终width之后，就对依赖RelativeLayout的子view
		//加上偏移量
            if (offsetHorizontalAxis) {
                for (int i = 0; i < count; i++) {
                    View child = getChildAt(i);
                    if (child.getVisibility() != GONE) {
                        LayoutParams params = (LayoutParams) child.getLayoutParams();
                        final int[] rules = params.getRules(layoutDirection);
                        //先对PARENT或者HORIZONTAL的子view重测
                        if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_HORIZONTAL] != 0) {
                            centerHorizontal(child, params, width);
                            //然后对ALIGN_PARENT_RIGHT重测
                        } else if (rules[ALIGN_PARENT_RIGHT] != 0) {
                            final int childWidth = child.getMeasuredWidth();
                            params.mLeft = width - mPaddingRight - childWidth;
                            params.mRight = params.mLeft + childWidth;
                        }
                    }
                }
            }
        }
	
	//这部分代码和width一样，就不多说了
        if (isWrapContentHeight) {
            // Height already has top padding in it since it was calculated by looking at
            // the bottom of each child view
            height += mPaddingBottom;

            if (mLayoutParams != null && mLayoutParams.height >= 0) {
                height = Math.max(height, mLayoutParams.height);
            }

            height = Math.max(height, getSuggestedMinimumHeight());
            height = resolveSize(height, heightMeasureSpec);

            if (offsetVerticalAxis) {
                for (int i = 0; i < count; i++) {
                    View child = getChildAt(i);
                    if (child.getVisibility() != GONE) {
                        LayoutParams params = (LayoutParams) child.getLayoutParams();
                        final int[] rules = params.getRules(layoutDirection);
                        if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_VERTICAL] != 0) {
                            centerVertical(child, params, height);
                        } else if (rules[ALIGN_PARENT_BOTTOM] != 0) {
                            final int childHeight = child.getMeasuredHeight();
                            params.mTop = height - mPaddingBottom - childHeight;
                            params.mBottom = params.mTop + childHeight;
                        }
                    }
                }
            }
        }
```

然后根据gravity对偏移量再一次修正

```
if (horizontalGravity || verticalGravity) {
            final Rect selfBounds = mSelfBounds;
            selfBounds.set(mPaddingLeft, mPaddingTop, width - mPaddingRight,
                    height - mPaddingBottom);

            final Rect contentBounds = mContentBounds;
            Gravity.apply(mGravity, right - left, bottom - top, selfBounds, contentBounds,
                    layoutDirection);

            final int horizontalOffset = contentBounds.left - left;
            final int verticalOffset = contentBounds.top - top;
            if (horizontalOffset != 0 || verticalOffset != 0) {
                for (int i = 0; i < count; i++) {
                    View child = getChildAt(i);
                    if (child.getVisibility() != GONE && child != ignore) {
                        LayoutParams params = (LayoutParams) child.getLayoutParams();
                        if (horizontalGravity) {
                            params.mLeft += horizontalOffset;
                            params.mRight += horizontalOffset;
                        }
                        if (verticalGravity) {
                            params.mTop += verticalOffset;
                            params.mBottom += verticalOffset;
                        }
                    }
                }
            }
        }
```

然后如果是右到左显示，再做一次修改

```
if (isLayoutRtl()) {
            final int offsetWidth = myWidth - width;
            for (int i = 0; i < count; i++) {
                View child = getChildAt(i);
                if (child.getVisibility() != GONE) {
                    LayoutParams params = (LayoutParams) child.getLayoutParams();
                    params.mLeft -= offsetWidth;
                    params.mRight -= offsetWidth;
                }
            }

        }
```

整个onMeasure()过程就结束了

看完一定会感觉头晕目眩，我自己再分析的时候都晕了好几次，大家需要耐心的在纸上把整个流程绘制下，耐心分析下，一定会对RelativeLayout有一个新的理解

最后再来看下流程

1.先把内部子view根据纵向关系和横向关系排序
2.初始化一些变量值
3.遍历水平关系的view
4.遍历竖直关系的view
5.baseline计算
6.宽度和高度修正

**总结**

对于LinearLayout来说，在measure的时候，只关心的子view的大小，所以只要遍历一次，如果有空余空间，可以根据weight的份额再遍历一次，分配大小

但是对于RelativeLayout来说，它更关心子view的left,right,bottom和top的值，这4个值的优先级是高于width和height的。如果在同时指定，并且有冲突的时候，优先采用的是left,right,bottom和top的值。
这样的设计思路也是有道理的，因为对RelativeLayout来说，子view的坐标都是相对的，只有找到那些rootView（不受其他view影响的view）然后再逐步确定别的view的坐标

最后来一个小例子证明下

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:gravity="bottom"
    android:layout_height="match_parent">


    <TextView
        android:layout_alignParentTop="true"
        android:id="@+id/top"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:background="@android:color/holo_orange_dark" />

    <TextView
        android:id="@+id/bottom"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:layout_alignParentBottom="true"
        android:background="@android:color/holo_blue_dark" />

    <TextView
        android:layout_below="@+id/top"
        android:layout_above="@+id/bottom"
        android:background="@android:color/holo_red_dark"
        android:id="@+id/middle"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />



</RelativeLayout>
```

这里分为3部分，上中下。
上下两部分的大小是指定的
中部view是红的，并且是在RelativeLayout的最下面（因为RelativeLayout上面的view是放在下面的view的下层，为了防止别的view干扰中部的红色view，所以放在最下面）

如果不指定
 android:layout_below="@+id/top"
        android:layout_above="@+id/bottom"
        这两个属性，那么中部view会充满全部屏幕

但是加上这两个属性之后，就会根据相对关系，计算出left,right,top,bottom属性然后把中间的view夹在上下两部分之间



**写在最后**

看源代码永远是进步最快的方式，新手的时候看源码，对于功能的实现仰望；进阶一点的时候对于源码的一些设计模式，逻辑处理感叹；即使达到专家层次再看一遍源码，会对于它的架构是如此的清晰，对自己的架构可以产生深远的影响。


		
