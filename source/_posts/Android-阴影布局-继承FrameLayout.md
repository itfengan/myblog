---
title: Android　阴影布局(继承FrameLayout)
date: 2017-03-15 17:52:55
tags: Android
---
# 阴影布局(继承FrameLayout) #

前言：
很多情况下，美腻的UI美眉喜欢搞一些花里胡哨阴影什么的,作为一名有追求的程序员迎合美眉的需求，搞一些小阴影并不是什么大问题，比如写一个自定义shape，用5.0的ｚ轴新特性和CardView都可以满足的，但是有些效果不太符合预计设计的效果，像自定义shape作为背景，看起来阴影会有些假，用５．０新特性第一个是版本问题还一个是有时候不起作用，网上也有解决不起作用的方法，我试了，都不太起作用，用cardview的话，如果cardview包裹的太多太复杂的控件，效果也不是太明显，所以有一个自定义FrameLayout来自己画阴影，以后再碰见阴影就又多了一种手段，满足应付设计师

<!--more-->

[https://github.com/itfengan/xShadowLayout](https://github.com/itfengan/xShadowLayout)

效果图

![这里写图片描述](http://img.blog.csdn.net/20171010160954830?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmVuZ2FuaXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```
package fengan.shadowdemo;

/**
 * Created by fengan on 2017/10/10/010.
 */


import android.annotation.SuppressLint;
import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Bitmap;
import android.graphics.BlurMaskFilter;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.PorterDuff;
import android.graphics.Rect;
import android.support.annotation.FloatRange;
import android.util.AttributeSet;
import android.widget.FrameLayout;

/**
 * Created by fengan on 10.10.2017.
 */
public class ShadowLayout extends FrameLayout {

    // Default shadow values
    private final static float DEFAULT_SHADOW_RADIUS = 30.0F;
    private final static float DEFAULT_SHADOW_DISTANCE = 15.0F;
    private final static float DEFAULT_SHADOW_ANGLE = 45.0F;
    private final static int DEFAULT_SHADOW_COLOR = Color.DKGRAY;

    // Shadow bounds values
    private final static int MAX_ALPHA = 255;
    private final static float MAX_ANGLE = 360.0F;
    private final static float MIN_RADIUS = 0.1F;
    private final static float MIN_ANGLE = 0.0F;
    // Shadow paint
    private final Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG) {
        {
            setDither(true);
            setFilterBitmap(true);
        }
    };
    // Shadow bitmap and canvas
    private Bitmap mBitmap;
    private final Canvas mCanvas = new Canvas();
    // View bounds
    private final Rect mBounds = new Rect();
    // Check whether need to redraw shadow
    private boolean mInvalidateShadow = true;

    // Detect if shadow is visible
    private boolean mIsShadowed;

    // Shadow variables
    private int mShadowColor;
    private int mShadowAlpha;
    private float mShadowRadius;
    private float mShadowDistance;
    private float mShadowAngle;
    private float mShadowDx;
    private float mShadowDy;

    public ShadowLayout(final Context context) {
        this(context, null);
    }

    public ShadowLayout(final Context context, final AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ShadowLayout(final Context context, final AttributeSet attrs, final int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        setWillNotDraw(false);
        setLayerType(LAYER_TYPE_HARDWARE, mPaint);

        // Retrieve attributes from xml
        final TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.ShadowLayout);
        try {
            setIsShadowed(typedArray.getBoolean(R.styleable.ShadowLayout_fengan_shadowed, true));
            setShadowRadius(
                    typedArray.getDimension(
                            R.styleable.ShadowLayout_fengan_shadow_radius, DEFAULT_SHADOW_RADIUS
                    )
            );
            setShadowDistance(
                    typedArray.getDimension(
                            R.styleable.ShadowLayout_fengan_shadow_distance, DEFAULT_SHADOW_DISTANCE
                    )
            );
            setShadowAngle(
                    typedArray.getInteger(
                            R.styleable.ShadowLayout_fengan_shadow_angle, (int) DEFAULT_SHADOW_ANGLE
                    )
            );
            setShadowColor(
                    typedArray.getColor(
                            R.styleable.ShadowLayout_fengan_shadow_color, DEFAULT_SHADOW_COLOR
                    )
            );
        } finally {
            typedArray.recycle();
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        // Clear shadow bitmap
        if (mBitmap != null) {
            mBitmap.recycle();
            mBitmap = null;
        }
    }

    public boolean isShadowed() {
        return mIsShadowed;
    }

    public void setIsShadowed(final boolean isShadowed) {
        mIsShadowed = isShadowed;
        postInvalidate();
    }

    public float getShadowDistance() {
        return mShadowDistance;
    }

    public void setShadowDistance(final float shadowDistance) {
        mShadowDistance = shadowDistance;
        resetShadow();
    }

    public float getShadowAngle() {
        return mShadowAngle;
    }

    @SuppressLint("SupportAnnotationUsage")
    @FloatRange
    public void setShadowAngle(@FloatRange(from = MIN_ANGLE, to = MAX_ANGLE) final float shadowAngle) {
        mShadowAngle = Math.max(MIN_ANGLE, Math.min(shadowAngle, MAX_ANGLE));
        resetShadow();
    }

    public float getShadowRadius() {
        return mShadowRadius;
    }

    public void setShadowRadius(final float shadowRadius) {
        mShadowRadius = Math.max(MIN_RADIUS, shadowRadius);

        if (isInEditMode()) return;
        // Set blur filter to paint
        mPaint.setMaskFilter(new BlurMaskFilter(mShadowRadius, BlurMaskFilter.Blur.NORMAL));
        resetShadow();
    }

    public int getShadowColor() {
        return mShadowColor;
    }

    public void setShadowColor(final int shadowColor) {
        mShadowColor = shadowColor;
        mShadowAlpha = Color.alpha(shadowColor);

        resetShadow();
    }

    public float getShadowDx() {
        return mShadowDx;
    }

    public float getShadowDy() {
        return mShadowDy;
    }

    // Reset shadow layer
    private void resetShadow() {
        // Detect shadow axis offset
        mShadowDx = (float) ((mShadowDistance) * Math.cos(mShadowAngle / 180.0F * Math.PI));
        mShadowDy = (float) ((mShadowDistance) * Math.sin(mShadowAngle / 180.0F * Math.PI));

        // Set padding for shadow bitmap
        final int padding = (int) (mShadowDistance + mShadowRadius);
        setPadding(padding, padding, padding, padding);
        requestLayout();
    }

    private int adjustShadowAlpha(final boolean adjust) {
        return Color.argb(
                adjust ? MAX_ALPHA : mShadowAlpha,
                Color.red(mShadowColor),
                Color.green(mShadowColor),
                Color.blue(mShadowColor)
        );
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        // Set ShadowLayout bounds
        mBounds.set(
                0, 0, MeasureSpec.getSize(widthMeasureSpec), MeasureSpec.getSize(heightMeasureSpec)
        );
    }

    @Override
    public void requestLayout() {
        // Redraw shadow
        mInvalidateShadow = true;
        super.requestLayout();
    }

    @Override
    protected void dispatchDraw(final Canvas canvas) {
        // If is not shadowed, skip
        if (mIsShadowed) {
            // If need to redraw shadow
            if (mInvalidateShadow) {
                // If bounds is zero
                if (mBounds.width() != 0 && mBounds.height() != 0) {
                    // Reset bitmap to bounds
                    mBitmap = Bitmap.createBitmap(
                            mBounds.width(), mBounds.height(), Bitmap.Config.ARGB_8888
                    );
                    // Canvas reset
                    mCanvas.setBitmap(mBitmap);

                    // We just redraw
                    mInvalidateShadow = false;
                    // Main feature of this lib. We create the local copy of all content, so now
                    // we can draw bitmap as a bottom layer of natural canvas.
                    // We draw shadow like blur effect on bitmap, cause of setShadowLayer() method of
                    // paint does`t draw shadow, it draw another copy of bitmap
                    super.dispatchDraw(mCanvas);

                    // Get the alpha bounds of bitmap
                    final Bitmap extractedAlpha = mBitmap.extractAlpha();
                    // Clear past content content to draw shadow
                    mCanvas.drawColor(0, PorterDuff.Mode.CLEAR);

                    // Draw extracted alpha bounds of our local canvas
                    mPaint.setColor(adjustShadowAlpha(false));
                    mCanvas.drawBitmap(extractedAlpha, mShadowDx, mShadowDy, mPaint);

                    // Recycle and clear extracted alpha
                    extractedAlpha.recycle();
                } else {
                    // Create placeholder bitmap when size is zero and wait until new size coming up
                    mBitmap = Bitmap.createBitmap(1, 1, Bitmap.Config.RGB_565);
                }
            }

            // Reset alpha to draw child with full alpha
            mPaint.setColor(adjustShadowAlpha(true));
            // Draw shadow bitmap
            if (mCanvas != null && mBitmap != null && !mBitmap.isRecycled())
                canvas.drawBitmap(mBitmap, 0.0F, 0.0F, mPaint);
        }

        // Draw child`s
        super.dispatchDraw(canvas);
    }
}

```
布局文件

```
 <?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="fengan.shadowdemo.MainActivity">
    <fengan.shadowdemo.ShadowLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:fengan_shadowed="true"
        app:fengan_shadow_angle="45"
        android:layout_centerInParent="true"
        app:fengan_shadow_radius="6dp"
        app:fengan_shadow_distance="10dp"
        app:fengan_shadow_color="#883F51B5">
        <TextView
            android:layout_width="250dp"
            android:layout_height="250dp"
            android:background="@drawable/bg"
            android:gravity="center"
            android:text="Hello World!"
            android:textColor="#ffffff"
            android:textSize="19sp"/>
    </fengan.shadowdemo.ShadowLayout>
</RelativeLayout>

```
自定义属性ａｔｔｒｓ．ｘｍｌ
```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="ShadowLayout">
        <attr name="fengan_shadowed" format="boolean"/>
        <attr name="fengan_shadow_distance" format="dimension"/>
        <attr name="fengan_shadow_angle" format="integer"/>
        <attr name="fengan_shadow_radius" format="dimension"/>
        <attr name="fengan_shadow_color" format="color"/>
    </declare-styleable>

</resources>
```
有灵性的哥哥们，已经猜到这些属性对应的意思啦．．．
在此整理方便大家日后使用．．．

