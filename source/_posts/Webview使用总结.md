---
title: Webview使用总结
date: 2017-03-12 16:00:27
tags: 
- Android
categories: webview
password: 123456
---

### WebView使用总结

**前言：**

最近，修改了项目中网页中点击图片跳转查看大图页面的需求，激发了我归纳总结WebView的想法，今天再次整理一下。

包含：使用过程，关键类，js交互，注意事项，以及几个常见的需求解决思路等。

<!--more-->

献上一张框架图

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/kd4ef.jpg)



### 基本简介

#### 内核

在4.4以前的版本是WebKit的内核，4.4以后才换成chromium的内核

可使用第三方WebView组件Crosswalk和TBS服务，具体区别后面再整理

#### 作用

- 显示和渲染web页面
- 直接加载Html文件（网络或者本地asserts）
- 与js互调

#### 状态

- webView.onResume() ；

激活WebView为活跃状态，能正常执行网页的响应

- webView.onPause()；

当页面被失去焦点被切换到后台不可见状态，需要执行onPause，通过onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。

- webView.pauseTimers()

[Webview后台耗电的问题](http://blog.csdn.net/mingli198611/article/details/49385557)

当应用程序(存在webview)被切换到后台时，需要暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。

可以调用此方法

- webView.resumeTimers()

恢复pauseTimers状态

**注意：**

由于pauseTimers和resumeTimers是全局生效的, 并不只影响单个WebView, resumeTimers注意不要遗漏, 否则遗漏的WebView会出现异常. 最好在重写的WebView

- rootLayout.removeView(webView); 
- webView.destroy();

销毁Webview，在关闭了Activity时，如果Webview的音乐或视频，还在播放。就必须销毁Webview

但是注意：webview调用destory时,webview仍绑定在Activity上

这是由于自定义webview构建时传入了该Activity的context对象

因此需要先从父容器中移除webview,然后再销毁webview:

#### 前进回退网页

- Webview.canGoBack() 

是否可以后退

- Webview.goBack()

后退网页

- Webview.canGoForward()

是否可以前进 

- Webview.goForward()

前进网页

- Webview.goBackOrForward(intsteps) 

以当前的index为起始点前进或者后退到历史记录中指定的steps

如果steps为负数则为后退，正数则为前进

**常见用法：**

- 问题

在不做任何处理前提下 ，浏览网页时点击系统的“Back”键,整个 Browser 会调用 finish()而结束自身

- 目标

点击返回后，是网页回退而不是推出浏览器

- 解决方案

在当前Activity中处理并消费掉该 Back 事件

```java
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if ((keyCode == KEYCODE_BACK) && mWebView.canGoBack()) { 
        mWebView.goBack();
        return true;
    }
    return super.onKeyDown(keyCode, event);
}
```



### 常用类

#### WebSettings

```java
//声明WebSettings子类
WebSettings webSettings = webView.getSettings();

//如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript
webSettings.setJavaScriptEnabled(true);  

//支持插件
webSettings.setPluginsEnabled(true); 

//设置自适应屏幕，两者合用
webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小 
webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小

//缩放操作
webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件

//其他细节操作
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); //关闭webview中缓存 
webSettings.setAllowFileAccess(true); //设置可以访问文件 
webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口 
webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片
webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式
//优先使用缓存: 
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); 
        //缓存模式如下：
        //LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
        //LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
        //LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
        //LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。
```

**常见缓存用法**

```java
if (NetStatusUtil.isConnected(getApplicationContext())) {
    webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);//根据cache-control决定是否从网络上取数据。
} else {
    webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);//没网，则从本地获取，即离线加载
}

webSettings.setDomStorageEnabled(true); // 开启 DOM storage API 功能
webSettings.setDatabaseEnabled(true);   //开启 database storage API 功能
webSettings.setAppCacheEnabled(true);//开启 Application Caches 功能

String cacheDirPath = getFilesDir().getAbsolutePath() + APP_CACAHE_DIRNAME;
webSettings.setAppCachePath(cacheDirPath); //设置  Application Caches 缓存目录
```

**每个 Application 只调用一次 WebSettings.setAppCachePath()，WebSettings.setAppCacheMaxSize()**

#### WebViewClient

##### **主要辅助WebView处理各种通知、请求事件**

```java
new WebViewClient(){
//            使得打开网页时不调用系统浏览器， 而是在本WebView中显示
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                return super.shouldOverrideUrlLoading(view, request);
            }

//            开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。
            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
            }

//            在页面加载结束时调用。我们可以关闭loading 条，切换程序动作。
            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
            }

//            在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。
            @Override
            public void onLoadResource(WebView view, String url) {
                super.onLoadResource(view, url);
            }

//            该方法传回了错误码，根据错误类型可以进行不同的错误分类处理
            @Override
            public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
                super.onReceivedError(view, request, error);
            }

//            webView默认是不处理https请求的，页面显示空白，需要进行如下设置：
            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
//                super.onReceivedSslError(view, handler, error);
                handler.proceed();    //表示等待证书响应
                // handler.cancel();      //表示挂起连接，为默认方式
                // handler.handleMessage(null);    //可做其他处理
            }
        };
```



#### WebChromClient

**辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等。**

- 获得网页的加载进度并显示

```java
webview.setWebChromeClient(new WebChromeClient(){

      @Override
      public void onProgressChanged(WebView view, int newProgress) {
          if (newProgress < 100) {
              String progress = newProgress + "%";
              progress.setText(progress);
            } else {
        }
    });
```

- **onReceivedTitle（）**

```java
webview.setWebChromeClient(new WebChromeClient(){

    @Override
    public void onReceivedTitle(WebView view, String title) {
       titleview.setText(title)；
    }
```

### JS交互

[原文](http://blog.csdn.net/carson_ho/article/details/64904691)

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/o45jt.png)

#### Android调Js

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/1m6qx.png)

**使用建议**

两种方法混合使用，即Android 4.4以下使用方法1，Android 4.4以上方法2

```java
// Android版本变量
final int version = Build.VERSION.SDK_INT;
// 因为该方法在 Android 4.4 版本才可使用，所以使用时需进行版本判断
if (version < 18) {
    mWebView.loadUrl("javascript:callJS()");
} else {
    mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
}
```

##### 通过loadUrl方式调用js

Html代码

```html
// 文本名：javascript
<!DOCTYPE html>
<html>

   <head>
      <meta charset="utf-8">
      <title>Carson_Ho</title>

// JS代码
     <script>
// Android需要调用的方法
   function callJS(){
      alert("Android调用了JS的callJS方法");
   }
</script>

   </head>

</html>
```

Android代码

```java
 // 设置与Js交互的权限
        webSettings.setJavaScriptEnabled(true);
        // 设置允许JS弹窗
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

        // 先载入JS代码
        // 格式规定为:file:///android_asset/文件名.html
        mWebView.loadUrl("file:///android_asset/javascript.html");

        button = (Button) findViewById(R.id.button);


        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 必须另开线程进行JS方法调用(否则无法调用)
                mWebView.post(new Runnable() {
                    @Override
                    public void run() {

                        // 注意调用的JS方法名要对应上
                        // 调用javascript的callJS()方法
                        mWebView.loadUrl("javascript:callJS()");
                    }
                });

            }
        });

        // 由于设置了弹窗检验调用结果,所以需要支持js对话框
        // webview只是载体，内容的渲染需要使用webviewChromClient类去实现
        // 通过设置WebChromeClient对象处理JavaScript的对话框
        //设置响应js 的Alert()函数
        mWebView.setWebChromeClient(new WebChromeClient() {
            @Override
            public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
                AlertDialog.Builder b = new AlertDialog.Builder(MainActivity.this);
                b.setTitle("Alert");
                b.setMessage(message);
                b.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        result.confirm();
                    }
                });
                b.setCancelable(false);
                b.create().show();
                return true;
            }

        });


    }
```

**特别注意：JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用。**

`onPageFinished()`属于WebViewClient类的方法，主要在页面加载结束时调用

所以一般等onPageFnished回调之后，我们才显示H5的页面，加载未完全的时候，则显示loading页面

##### 通过evaluateJavascript调用js

**优点**

该方法比第一种方法效率更高、使用更简洁。

- 因为该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会。
- Android 4.4 后才可使用

```java
// 只需要将第一种方法的loadUrl()换成下面该方法即可
    mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
}
```



#### Js调Android

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/vhjwh.png)

### 缓存问题

[原文](https://www.jianshu.com/p/f1efb0928ebc)

### 参考链接

[Carson_Ho](http://blog.csdn.net/carson_ho/article/details/52693322)

