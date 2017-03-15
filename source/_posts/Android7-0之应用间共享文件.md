---
title: Android7.0之应用间共享文件
date: 2017-01-23 16:58:17
tags: Android
---
## Android7.0适配之应用间共享文件 ##


### 出现问题 ###
Android N(API 25),打开相册编辑页面crash,报出FileUriExposedException异常

```
    android.os.FileUriExposedException: file:////storage/emulated/0/temp/1474956193735.jpg exposed beyond app through Intent.getData()
at android.os.StrictMode.onFileUriExposed(StrictMode.java:1799)
at android.net.Uri.checkFileUriExposed(Uri.java:2346)
at android.content.Intent.prepareToLeaveProcess(Intent.java:8933)
at android.content.Intent.prepareToLeaveProcess(Intent.java:8894)
at android.app.Instrumentation.execStartActivity(Instrumentation.java:1517)
at android.app.Activity.startActivityForResult(Activity.java:4223)
...
at android.app.Activity.startActivityForResult(Activity.java:4182)
```
<!--more-->
### 查找原因 ###
Android N的应用,API禁止向应用外公开file://URI,如果一项包含文件URI的Intent离开应用, 应用crash并报FileUriExposedException异常
### 解决办法 ###
若想要在应用间共享文件,应该发送一项content://URI,并该URI临时访问权限,进行此授权的方式是通过FileProvider类 
### 具体步骤 ###
#### [1] 清单文件 ####
```
    <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.fengan.providerdemo">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
           <!--authorities="你的包名+fileprovider" -->
        <provider
            android:authorities="com.fengan.providerdemo.fileprovider"
            android:name="android.support.v4.content.FileProvider"
            android:grantUriPermissions="true"
            android:exported="false">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths"/>
        </provider>
    </application>

</manifest>
```
#### [2] res下xml文件夹 ####
![Alt text](./1485158383668.png)
注意xml文件名和清单文件中@xml/filepaths相同
xml文件内容

```
    <?xml version="1.0" encoding="utf-8"?>
<paths>
    <!-- external-path:sd ；path:你的应用保存文件的根目录；name随便定义-->
    //<external-path path="fengan_imgs/" name="files_path" />
    <external-path path="" name="files_path" />
</paths>
```
注意:
path="",有特殊意义,它代表更目录,也就是说可以向应用共享根目录及其子目录下任何一个文件,如果将path写为path="fengan_imgs/",那么只能在fengan_imgs/目录下才可以分享!
[3]核心代码
将File转换为uri

```
 private static Uri getUriForFile(Context context, File file) {
        if (context == null || file == null) {
            throw new NullPointerException();
        }
        Uri uri;
        if (Build.VERSION.SDK_INT >= 24) {
        //和android:authorities="com.fengan.providerdemo.fileprovider"对应
            uri = FileProvider.getUriForFile(context.getApplicationContext(), "com.fengan.providerdemo.fileprovider", file);
        } else {
            uri = Uri.fromFile(file);
        }
        return uri;
    }
```




- Uri的scheme类型为file,改成了又FileProvider创建一个content类型的Uri打开相机,打印该Uri为content://com.fengan.providerdemo/files_path/temp/1474960080319.jpg`。 
//其中camera_photos就是file_paths.xml中paths的name。

```
 /**
     * 打开相机
     * 兼容7.0
     *
     * @param activity    Activity
     * @param file        File
     * @param requestCode result requestCode
     */
    public static void startActionCapture(Activity activity, File file, int requestCode) {
        if (activity == null) {
            return;
        }
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
     intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION); //添加这一句表示对目标应用临时授权该Uri所代表的文件
        intent.putExtra(MediaStore.EXTRA_OUTPUT, getUriForFile(activity, file));//拍取照片保存到指定Uri
        activity.startActivityForResult(intent, requestCode);
    }
```
## 总结 ##
 - 针对涉及到从Android设备上获取照片(拍照,或从相册,文件中选择)打开相机,裁剪图片,压缩图片,可以使用一个轻量级开源库,TakePhoto!
 https://github.com/crazycodeboy/TakePhoto/

