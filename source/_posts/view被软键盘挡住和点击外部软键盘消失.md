---
title: view被软键盘挡住和点击外部软键盘消失
date: 2016-08-21 11:02:01
tags: 
- Android
categories: Android
---
软键盘遮挡和点击空白区域关闭键盘

<!--more-->

### 需求

- 登陆button挡住输入框
- 软键盘弹出挡住其他控件
- 软键盘弹出，点击空白区域关闭软件盘


#### 点击button，键盘将button顶上去####


> ![引用块内容](http://img.blog.csdn.net/20170615102440904?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmVuZ2FuaXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


    //    控制是否移动布局。比如只有密码输入框获取到焦点时才执行。
    public  boolean flag = true;
    
    /**
     * @param act          activiry用于获取底部导航栏高度。
     * @param root         最外层布局，需要调整的布局
     * @param scrollToView 被键盘遮挡的scrollToView，滚动root,使scrollToView在root可视区域的底部
     */
    public  void controlKeyboardLayout(Context act, final View root, final View scrollToView) {
        final int navigationBarHeight = getNavigationBarHeight(act);
    
        root.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                Rect rect = new Rect();
                //获取root在窗体的可视区域
                root.getWindowVisibleDisplayFrame(rect);
                //获取root在窗体的不可视区域高度(被其他View遮挡的区域高度)
                int rootInvisibleHeight = root.getRootView().getHeight() - rect.bottom;
                //若不可视区域高度大于100，则键盘显示
                if (rootInvisibleHeight > navigationBarHeight && flag) {
                    int[] location = new int[2];
                    //获取scrollToView在窗体的坐标
                    scrollToView.getLocationInWindow(location);
                    //计算root滚动高度，使scrollToView在可见区域
                    int srollHeight = (location[1] + scrollToView.getHeight()) - rect.bottom;
                    if (root.getScrollY() != 0) {// 如果已经滚动，要根据上次滚动，重新计算位置。
                        srollHeight += root.getScrollY();
                    }
                    root.scrollTo(0, srollHeight);
                } else {
                    //键盘隐藏
                    root.scrollTo(0, 0);
                }
            }
        });
    }

### 解决控制点击可选择的区域让软键盘消失或者不消失 ###
      /**
     * 获取底部导航栏高度
     *
     * @param act
     * @return
     */
    private  int getNavigationBarHeight(Context act) {
        Resources resources = act.getResources();
        int resourceId = resources.getIdentifier("navigation_bar_height", "dimen", "android");
        int height = resources.getDimensionPixelSize(resourceId);
        Log.v("dbw", "Navi height:" + height);
        return height;
    }
    
    //软键盘消失的管理
    //region软键盘的处理
    
    /**
     * 清除editText的焦点
     *
     * @param v   焦点所在View
     * @param ids 输入框
     */
    public void clearViewFocus(View v, int... ids) {
        if (null != v && null != ids && ids.length > 0) {
            for (int id : ids) {
                if (v.getId() == id) {
                    v.clearFocus();
                    break;
                }
            }
        }
    }
    /**
     * 隐藏键盘
     *
     * @param v   焦点所在View
     * @param ids 输入框
     * @return true代表焦点在edit上
     */
    public boolean isFocusEditText(View v, int... ids) {
        if (v instanceof EditText) {
            EditText tmp_et = (EditText) v;
            for (int id : ids) {
                if (tmp_et.getId() == id) {
                    return true;
                }
            }
        }
        return false;
    }
    
    //是否触摸在指定view上面,对某个控件过滤
    public boolean isTouchView(View[] views, MotionEvent ev) {
        if (views == null || views.length == 0) return false;
        int[] location = new int[2];
        for (View view : views) {
            view.getLocationOnScreen(location);
            int x = location[0];
            int y = location[1];
            if (ev.getX() > x && ev.getX() < (x + view.getWidth())
                    && ev.getY() > y && ev.getY() < (y + view.getHeight())) {
                return true;
            }
        }
        return false;
    }
    
    //region 右滑返回上级
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            if (isTouchView(filterViewByIds(), ev)) return super.dispatchTouchEvent(ev);
            if (hideSoftByEditViewIds() == null || hideSoftByEditViewIds().length == 0)
                return super.dispatchTouchEvent(ev);
            View v = getCurrentFocus();
            if (isFocusEditText(v, hideSoftByEditViewIds())) {
                //隐藏键盘
                hideInputForce(this);
                clearViewFocus(v, hideSoftByEditViewIds());
            }
        }
        return super.dispatchTouchEvent(ev);
    
    }


    /**
     * 传入EditText的Id
     * 没有传入的EditText不做处理
     *
     * @return id 数组
     */
    public int[] hideSoftByEditViewIds() {
        return null;
    }
    
    /**
     * 传入要过滤的View
     * 过滤之后点击将不会有隐藏软键盘的操作
     *
     * @return id 数组
     */
    public View[] filterViewByIds() {
        return null;
    }
    
    /**
     * des:隐藏软键盘,这种方式参数为activity
     *
     * @param activity
     */
    public static void hideInputForce(Activity activity) {
        if (activity == null || activity.getCurrentFocus() == null)
            return;
    
        ((InputMethodManager) activity.getSystemService(INPUT_METHOD_SERVICE))
                .hideSoftInputFromWindow(activity.getCurrentFocus()
                        .getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);
    }

### 以下Demo全部代码 ###


```java
public class MainActivity extends AppCompatActivity {
private EditText mEt1;
private EditText mEt2;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Button btn = (Button) findViewById(R.id.btn);
    mEt1 = (EditText) findViewById(R.id.et1);
    mEt2 = (EditText) findViewById(R.id.et2);
    LinearLayout LL = (LinearLayout) findViewById(R.id.LL);
    controlKeyboardLayout(this,LL,btn);
}
```


```java
//    控制是否移动布局。比如只有密码输入框获取到焦点时才执行。
public  boolean flag = true;

/**
 * @param act          activiry用于获取底部导航栏高度。
 * @param root         最外层布局，需要调整的布局
 * @param scrollToView 被键盘遮挡的scrollToView，滚动root,使scrollToView在root可视区域的底部
 */
public  void controlKeyboardLayout(Context act, final View root, final View scrollToView) {
    final int navigationBarHeight = getNavigationBarHeight(act);

    root.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
        @Override
        public void onGlobalLayout() {
            Rect rect = new Rect();
            //获取root在窗体的可视区域
            root.getWindowVisibleDisplayFrame(rect);
            //获取root在窗体的不可视区域高度(被其他View遮挡的区域高度)
            int rootInvisibleHeight = root.getRootView().getHeight() - rect.bottom;
            //若不可视区域高度大于100，则键盘显示
            if (rootInvisibleHeight > navigationBarHeight && flag) {
                int[] location = new int[2];
                //获取scrollToView在窗体的坐标
                scrollToView.getLocationInWindow(location);
                //计算root滚动高度，使scrollToView在可见区域
                int srollHeight = (location[1] + scrollToView.getHeight()) - rect.bottom;
                if (root.getScrollY() != 0) {// 如果已经滚动，要根据上次滚动，重新计算位置。
                    srollHeight += root.getScrollY();
                }
                root.scrollTo(0, srollHeight);
            } else {
                //键盘隐藏
                root.scrollTo(0, 0);
            }
        }
    });
}

/**
 * 获取底部导航栏高度
 *
 * @param act
 * @return
 */
private  int getNavigationBarHeight(Context act) {
    Resources resources = act.getResources();
    int resourceId = resources.getIdentifier("navigation_bar_height", "dimen", "android");
    int height = resources.getDimensionPixelSize(resourceId);
    Log.v("dbw", "Navi height:" + height);
    return height;
}

//软键盘消失的管理
//region软键盘的处理

/**
 * 清除editText的焦点
 *
 * @param v   焦点所在View
 * @param ids 输入框
 */
public void clearViewFocus(View v, int... ids) {
    if (null != v && null != ids && ids.length > 0) {
        for (int id : ids) {
            if (v.getId() == id) {
                v.clearFocus();
                break;
            }
        }
    }
}
/**
 * 隐藏键盘
 *
 * @param v   焦点所在View
 * @param ids 输入框
 * @return true代表焦点在edit上
 */
public boolean isFocusEditText(View v, int... ids) {
    if (v instanceof EditText) {
        EditText tmp_et = (EditText) v;
        for (int id : ids) {
            if (tmp_et.getId() == id) {
                return true;
            }
        }
    }
    return false;
}

//是否触摸在指定view上面,对某个控件过滤
public boolean isTouchView(View[] views, MotionEvent ev) {
    if (views == null || views.length == 0) return false;
    int[] location = new int[2];
    for (View view : views) {
        view.getLocationOnScreen(location);
        int x = location[0];
        int y = location[1];
        if (ev.getX() > x && ev.getX() < (x + view.getWidth())
                && ev.getY() > y && ev.getY() < (y + view.getHeight())) {
            return true;
        }
    }
    return false;
}

//region 右滑返回上级
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        if (isTouchView(filterViewByIds(), ev)) return super.dispatchTouchEvent(ev);
        if (hideSoftByEditViewIds() == null || hideSoftByEditViewIds().length == 0)
            return super.dispatchTouchEvent(ev);
        View v = getCurrentFocus();
        if (isFocusEditText(v, hideSoftByEditViewIds())) {
            //隐藏键盘
            hideInputForce(this);
            clearViewFocus(v, hideSoftByEditViewIds());
        }
    }
    return super.dispatchTouchEvent(ev);

}
```


```java
/**
 * 传入EditText的Id
 * 没有传入的EditText不做处理
 *
 * @return id 数组
 */
public int[] hideSoftByEditViewIds() {
    int []  ids = {R.id.et1,R.id.et2};
    return ids;
}

/**
 * 传入要过滤的View
 * 过滤之后点击将不会有隐藏软键盘的操作
 *
 * @return id 数组
 */
public View[] filterViewByIds() {
    View [] views = {mEt1,mEt2};//点击这两个控件,软键盘不会消失
    return views;
}

/**
 * des:隐藏软键盘,这种方式参数为activity
 *
 * @param activity
 */
public static void hideInputForce(Activity activity) {
    if (activity == null || activity.getCurrentFocus() == null)
        return;

    ((InputMethodManager) activity.getSystemService(INPUT_METHOD_SERVICE))
            .hideSoftInputFromWindow(activity.getCurrentFocus()
                    .getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);
}
}
```
### 以下是布局文件 ###

```Xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
android:id="@+id/activity_main"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical"
tools:context="fengan.softinputdemo.MainActivity">
<LinearLayout
    android:id="@+id/LL"
    android:layout_marginTop="100dp"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="wrap_content">
<EditText
    android:id="@+id/et1"
    android:layout_marginTop="60dp"
    android:background="#ff0"
    android:layout_width="match_parent"
    android:layout_height="60dp"/>
<EditText
    android:id="@+id/et2"
    android:layout_marginTop="20dp"
    android:background="#ff0"
    android:layout_width="match_parent"
    android:layout_height="60dp"/>
<Button
    android:text="软键盘挡住button"
    android:id="@+id/btn"
    android:layout_marginTop="30dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
</LinearLayout>
```


- 可以将隐藏显示的代码封装到BaseActivity


<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=448707059&auto=1&height=66"></iframe>