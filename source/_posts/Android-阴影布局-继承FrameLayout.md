---
title: Android　阴影布局(继承FrameLayout)
date: 2017-03-15 17:52:55
tags: Android
---
## 阴影布局(继承FrameLayout) ##

# 前言 #
很多情况下，坑爹的又矮射鸡屎为了炫耀他们的出神入化的设计，喜欢搞一些花里胡哨的东西，搞些阴影什么的．
作为一名有追求的程序员，搞一些小阴影并不是什么大问题，比如写一个自定义shape，用5.0的ｚ轴新特性和CardView都可以满足的，但是有些效果不太符合预计设计的效果，像自定义shape作为背景，看起来阴影会有些假，用５．０新特性第一个是版本问题还一个是有时候不起作用，网上也有解决不起作用的方法，我试了，都不太起作用，用cardview的话，如果cardview包裹的太多太复杂的控件，效果也不是太明显，所以有一个自定义FrameLayout来自己画阴影，以后再碰见阴影就又多了一种手段，满足应付设计师

不要搞一些花里胡哨的,黏贴过去以简单粗暴的方式直接用
# 开始你的表演 #
### 自定义view ###
    import android.content.Context;
	import android.content.res.TypedArray;
	import android.graphics.Bitmap;
	import android.graphics.Canvas;
	import android.graphics.Color;
	import android.graphics.Paint;
	import android.graphics.RectF;
	import android.graphics.drawable.BitmapDrawable;
	import android.os.Build;
	import android.util.AttributeSet;
	import android.widget.FrameLayout;

	import com.rz.rz_rrz.R;

	public class ShadowLayout extends FrameLayout {

    private int mShadowColor;
    private float mShadowRadius;
    private float mCornerRadius;
    private float mDx;
    private float mDy;

    private boolean mInvalidateShadowOnSizeChanged = true;
    private boolean mForceInvalidateShadow = false;

    public ShadowLayout(Context context) {
        super(context);
        initView(context, null);
    }

    public ShadowLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context, attrs);
    }

    public ShadowLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    @Override
    protected int getSuggestedMinimumWidth() {
        return 0;
    }

    @Override
    protected int getSuggestedMinimumHeight() {
        return 0;
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if(w > 0 && h > 0 && (getBackground() == null || mInvalidateShadowOnSizeChanged || mForceInvalidateShadow)) {
            mForceInvalidateShadow = false;
            setBackgroundCompat(w, h);
        }
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        if (mForceInvalidateShadow) {
            mForceInvalidateShadow = false;
            setBackgroundCompat(right - left, bottom - top);
        }
    }

    public void setInvalidateShadowOnSizeChanged(boolean invalidateShadowOnSizeChanged) {
        mInvalidateShadowOnSizeChanged = invalidateShadowOnSizeChanged;
    }

    public void invalidateShadow() {
        mForceInvalidateShadow = true;
        requestLayout();
        invalidate();
    }

    private void initView(Context context, AttributeSet attrs) {
        initAttributes(context, attrs);

        int xPadding = (int) (mShadowRadius + Math.abs(mDx));
        int yPadding = (int) (mShadowRadius + Math.abs(mDy));
        setPadding(xPadding, yPadding, xPadding, yPadding);
    }

    @SuppressWarnings("deprecation")
    private void setBackgroundCompat(int w, int h) {
        Bitmap bitmap = createShadowBitmap(w, h, mCornerRadius, mShadowRadius, mDx, mDy, mShadowColor, Color.TRANSPARENT);
        BitmapDrawable drawable = new BitmapDrawable(getResources(), bitmap);
        if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.JELLY_BEAN) {
            setBackgroundDrawable(drawable);
        } else {
            setBackground(drawable);
        }
    }


    private void initAttributes(Context context, AttributeSet attrs) {
        TypedArray attr = getTypedArray(context, attrs, R.styleable.ShadowLayout);
        if (attr == null) {
            return;
        }

        try {
            mCornerRadius = attr.getDimension(R.styleable.ShadowLayout_sl_cornerRadius, getResources().getDimension(R.dimen.default_corner_radius));
            mShadowRadius = attr.getDimension(R.styleable.ShadowLayout_sl_shadowRadius, getResources().getDimension(R.dimen.default_shadow_radius));
            mDx = attr.getDimension(R.styleable.ShadowLayout_sl_dx, 0);
            mDy = attr.getDimension(R.styleable.ShadowLayout_sl_dy, 0);
            mShadowColor = attr.getColor(R.styleable.ShadowLayout_sl_shadowColor, getResources().getColor(R.color.default_shadow_color));
        } finally {
            attr.recycle();
        }
    }

    private TypedArray getTypedArray(Context context, AttributeSet attributeSet, int[] attr) {
        return context.obtainStyledAttributes(attributeSet, attr, 0, 0);
    }

    private Bitmap createShadowBitmap(int shadowWidth, int shadowHeight, float cornerRadius, float shadowRadius,
                                      float dx, float dy, int shadowColor, int fillColor) {

        Bitmap output = Bitmap.createBitmap(shadowWidth, shadowHeight, Bitmap.Config.ALPHA_8);
        Canvas canvas = new Canvas(output);

        RectF shadowRect = new RectF(
                shadowRadius,
                shadowRadius,
                shadowWidth - shadowRadius,
                shadowHeight - shadowRadius);

        if (dy > 0) {
            shadowRect.top += dy;
            shadowRect.bottom -= dy;
        } else if (dy < 0) {
            shadowRect.top += Math.abs(dy);
            shadowRect.bottom -= Math.abs(dy);
        }

        if (dx > 0) {
            shadowRect.left += dx;
            shadowRect.right -= dx;
        } else if (dx < 0) {
            shadowRect.left += Math.abs(dx);
            shadowRect.right -= Math.abs(dx);
        }

        Paint shadowPaint = new Paint();
        shadowPaint.setAntiAlias(true);
        shadowPaint.setColor(fillColor);
        shadowPaint.setStyle(Paint.Style.FILL);

        if (!isInEditMode()) {
            shadowPaint.setShadowLayer(shadowRadius, dx, dy, shadowColor);
        }

        canvas.drawRoundRect(shadowRect, cornerRadius, cornerRadius, shadowPaint);

        return output;
    }

}

布局文件

        <com.rz.rz_rrz.widget.ShadowLayout
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        app:sl_dx="1dp"
                        app:sl_dy="1dp"
                        ／／．．．自由发挥．．．
 	 	`</com.rz.rz_rrz.widget.ShadowLayout>`
自定义属性attrs.xml
```
    <declare-styleable name="ShadowLayout">
        <attr name="sl_cornerRadius" format="dimension"/>
        <attr name="sl_shadowRadius" format="dimension"/>
        <attr name="sl_shadowColor" format="color"/>
        <attr name="sl_dx" format="dimension"/>
        <attr name="sl_dy" format="dimension"/>
    </declare-styleable>
```
有灵性的哥哥们，已经猜到这些属性对应的意思啦．．．
在此整理方便大家日后使用．．．
