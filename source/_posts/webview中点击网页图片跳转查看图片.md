---
title: webview中点击网页图片跳转查看图片
date: 2016-03-08 16:11:40
tags: 
- Android
categories: Android
password: 123456
---

**前言**

app中很多图文页面，类似新闻详情页，都是一个H5网页，当我们点击网页中的图片，希望跳转到一个多图游览的页面，并且支持手势缩放和保存的一些功能。今天主要整理一下点击网页中图片，跳转到指定图片的查看页面即可，手势缩放可以使用photoview。

<!--more-->

### 原理

#### 效果

![网页点击图片](https://ws4.sinaimg.cn/large/006tNc79ly1fp5hh5mdkvg30bf0lgnpd.gif)

**原理**

首先点击h5页面，跳转本地页面，是js调用原生代码

**Js通过WebView调用Android代码有三种方式**

- 通过WebView的addJavascriptInterface（）进行对象映射
- 通过 WebViewClient 的shouldOverrideUrlLoading ()方法回调拦截 url
- 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截JS对话框alert()、confirm()、prompt（） 消息

![js调原生](https://ws4.sinaimg.cn/large/006tNc79ly1fp5hmm8ywuj31d60hqgqf.jpg)

通过addJavascriptInterface定义js接口的方式比较方便，所以本文采用的是这种方式，详细见下列代码

- 定义js接口
- 找到网页中img标签的代码块，设置点击
- 相关权限和混淆

### 代码

#### 定义js接口

```java
**
 * Created by fengan on 2016/3/8.
 * email:fengan1102@gmail.com
 * 点击webview页面图片，跳转查看大图页面的js接口
 * 检查混淆文件，确保未被混淆
 */

public class JavaScriptInterface {
    private Activity mContext;
    private ArrayList<String> mImgList = new ArrayList<>();

    public JavaScriptInterface(Activity context) {
        this.mContext = context;
    }

//方法名要和执行的js代码中一致
    @JavascriptInterface
    public void addImageUrl(String img) { 
        mImgList.add(img);
    }

    @JavascriptInterface
    public void openImage(String img) {
        if (ClickUtils.noDoubleClick()) {
            int index = mImgList.indexOf(img);
            PhotoListAty.startAty(index == -1 ? 0 : index, mImgList, mContext);
        }
    }
}

```

#### 自定义WebClient

```java
public class MyWebViewClient extends WebViewClient {  

    @Override  

    public void onPageFinished(WebView view, String url) {  

        view.getSettings().setJavaScriptEnabled(true);  

      super.onPageFinished(view, url);  

    addImageClickListener(view);//待网页加载完全后设置图片点击的监听方法  

    }  

 

  @Override  

   public void onPageStarted(WebView view, String url, Bitmap favicon) {  

        view.getSettings().setJavaScriptEnabled(true);  

       super.onPageStarted(view, url, favicon);  

   }  

 

    private void addImageClickListener(WebView webView) {  
		 webView.loadUrl("javascript:(function(){" +
                "var objs = document.getElementsByTagName(\"img\"); " +
                "for(var i=0;i<objs.length;i++)  " +
                "{" +
                "window.imagelistener.addImageUrl(objs[i].src);  " +
                " objs[i].onclick=function()  " +
                " {  " +
                " window.imagelistener.openImage(this.src);  " +
                "  }  " +
                "}" +
                "})()");
   }  

```

```java
//imagelistener用于暴露js的对象，需要对应
 mWebview.getSettings().setJavaScriptEnabled(true);  
 mWebview.addJavascriptInterface(new JavaScriptInterface(this), "imagelistener");
 mWebview.setWebViewClient(new MyWebViewClient());  
```



### 注意事项

注意在混淆文件中添加

项目中暴露的js接口类：MJavascriptInterface不能混淆，其调用的方法的声明也不能混淆，所以还要添加如下混淆设置代码（代码因包名而变化）

```properties
-keepclassmembers class com.example.administrator.webviewpagescannerapp.other.MJavascriptInterface{  
  public *;  
}  
  
-keepattributes *Annotation*  
-keepattributes *JavascriptInterface*
```