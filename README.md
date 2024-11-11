## [TakePhoto](https://github.com/crazycodeboy/TakePhoto) 简介

[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg)](https://github.com/mehroz-dev/TakePhoto/pulls)
[![GitHub release](https://img.shields.io/github/release/mehroz-dev/TakePhoto.svg?maxAge=2592000?style=flat-square)](https://github.com/mehroz-dev/TakePhoto/releases)
[![License Apache2.0](http://img.shields.io/badge/license-Apache2.0-brightgreen.svg?style=flat)](https://raw.githubusercontent.com/mehroz-dev/TakePhoto/master/LICENSE)



TakePhoto is an open-source tool library for Android devices that enables photo capture (taking a picture or selecting from the album or files), image cropping, and image compression. The latest version is currently available.[4.1.0](https://github.com/mehroz-dev/TakePhoto/)。See versions below 3.0 and the API documentation for details.[TakePhoto2.0+](https://github.com/mehroz-dev/TakePhoto/blob/master/README.2+.md)。  

>TakePhoto Community Platform: QQ Group: 556387607 (Group 1, not full)

V4.0

Supports capturing images via the camera
Supports selecting images from the album
Supports selecting images from files
Supports batch selection of images
Supports image compression and batch image compression
Supports image cropping and batch image cropping
Automatically corrects photo rotation angles
Supports automatic permission management (no need to worry about SD card or camera permissions)
Allows for personalized configuration of cropping and compression parameters
Provides a built-in cropping tool (optional)
Supports smart selection and handling of cropping exceptions
Supports automatic recovery if the Activity for capturing photos is recycled
Compatible with Android 8.1
Supports multiple compression tools
Supports multiple image selection tools

GitHub address: [https://github.com/mehroz-dev/TakePhoto](https://github.com/mehroz-dev/TakePhoto)
## Table of Contents

Installation Instructions
Demo
Usage Instructions
Custom UI
API
Compatibility
Contributions
Changelog
Final Notes
Installation Instructions

Gradle:

```groovy
    compile 'com.jph.takephoto:takephoto_library:4.1.0'
```

**Maven:**  

```groovy
<dependency>
  <groupId>com.jph.takephoto</groupId>
  <artifactId>takephoto_library</artifactId>
  <version>4.1.0</version>
  <type>pom</type>
</dependency>
```  



Usage Instructions
There are two ways to use TakePhoto:
Method 1: By inheritance

Inherit from one of the following: TakePhotoActivity, TakePhotoFragmentActivity, or TakePhotoFragment.
Use getTakePhoto() to obtain a TakePhoto instance and perform the necessary operations.
Override the following methods to get results.       

```java
 void takeSuccess(TResult result);
 void takeFail(TResult result,String msg);
 void takeCancel();
```  
This method is simple to use and meets most usage needs. For details, see[simple](https://github.com/mehroz-dev/TakePhoto/blob/master/simple/src/main/java/com/jph/simple/SimpleActivity.java)。If inheritance cannot meet the needs of an actual project, you can use the following method.

Method 2: Using Composition

Refer to the following：[TakePhotoActivity](https://github.com/mehroz-dev/TakePhoto/blob/master/takephoto_library/src/main/java/com/jph/takephoto/app/TakePhotoActivity.java)，The main steps are as follows:  

1. Implement the TakePhoto.TakeResultListener and InvokeListener interfaces.

2.Call the corresponding methods of TakePhoto in the onCreate, onActivityResult, and onSaveInstanceState methods.

3.Override onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) and add the following code.

```java
  @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        TPermissionType type=PermissionManager.onRequestPermissionsResult(requestCode,permissions,grantResults);
        PermissionManager.handlePermissionsResult(this,type,invokeParam,this);
    }
```    

4. Override the TPermissionType invoke(InvokeParam invokeParam) method and add the following code:  

```java
 @Override
    public TPermissionType invoke(InvokeParam invokeParam) {
        TPermissionType type=PermissionManager.checkPermission(TContextWrap.of(this),invokeParam.getMethod());
        if(TPermissionType.WAIT.equals(type)){
            this.invokeParam=invokeParam;
        }
        return type;
    }
```

5. Add the following code to obtain the TakePhoto instance:  

```java
   /**
     *  Obtain TakePhoto instance
     * @return
     */
    public TakePhoto getTakePhoto(){
        if (takePhoto==null){
            takePhoto= (TakePhoto) TakePhotoInvocationHandler.of(this).bind(new TakePhotoImpl(this,this));
        }
        return takePhoto;
    }    
```

Custom UI
TakePhoto not only supports customization of related parameters but also allows customization of the UI. Below is an introduction on how to customize the gallery and cropping tool UI of TakePhoto.

Custom Gallery
If the default gallery UI of TakePhoto does not match the theme of your application, you can customize it. The method is as follows:

Custom Toolbar
Create a layout file named "toolbar.xml" in the "res/layout" directory with the following content:：   

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:theme="@style/CustomToolbarTheme"
    android:background="#ffa352">
</android.support.v7.widget.Toolbar>
```

In the toolbar.xml file, you can specify the theme for the TakePhoto gallery and the background color of the Toolbar.

Custom Status Bar
Create a resource file named colors.xml in the res/values directory with the following content: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="multiple_image_select_primaryDark">#212121</color>
</resources>
```

By using the above method, you can customize the color of the status bar.

Custom Tip Text
In the string.xml file in the res/values directory, add the following code:

```xml
<resources>    
    <string name="album_view">Select Image</string>
    <string name="image_view">Click to Select</string>
    <string name="add">Confirm</string>
    <string name="selected">Selected</string>
    <string name="limit_exceeded">You can select up to %d images</string>
</resources>
```

By overriding the above code, you can customize the tip text in the TakePhoto gallery.

Custom Cropping Tool
Create layout files named crop__activity_crop.xml and crop__layout_done_cancel.xml in the res/layout directory with the following content:

crop__activity_crop.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <com.soundcloud.android.crop.CropImageView
        android:id="@+id/crop_image"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_alignParentTop="true"
        android:background="@drawable/crop__texture"
        android:layout_above="@+id/done_cancel_bar" />
    <include
        android:id="@+id/done_cancel_bar"
        android:layout_alignParentBottom="true"
        layout="@layout/crop__layout_done_cancel"
        android:layout_height="50dp"
        android:layout_width="match_parent" />
</RelativeLayout>
```

**crop__layout_done_cancel.xml**  

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    style="@style/Crop.DoneCancelBar">
    <FrameLayout
        android:id="@+id/btn_cancel"
        style="@style/Crop.ActionButton">
        <TextView style="@style/Crop.ActionButtonText.Cancel" />
    </FrameLayout>
    <FrameLayout
        android:id="@+id/btn_done"
        style="@style/Crop.ActionButton">
        <TextView style="@style/Crop.ActionButtonText.Done" />
    </FrameLayout>
</LinearLayout>
```

Override the above code to customize the UI of the TakePhoto cropping tool.

API
Get Image
TakePhoto provides three methods to obtain images: from the camera, from the gallery, and from files.

API:

```java
/**
 * Get image from file (no cropping)
 */
void onPickFromDocuments();
/**
 * Get image from gallery (no cropping)
 */
void onPickFromGallery();
/**
 * Get image from camera (no cropping)
 * @param outPutUri The path where the image will be saved
 */
void onPickFromCapture(Uri outPutUri);
/**
 * Multi-image selection
 * @param limit The maximum number of images that can be selected
 **/
void onPickMultiple(int limit);
```

The three methods above all provide corresponding cropping APIs.

Note:
Due to varying levels of system customization by different Android ROM manufacturers, some image selection methods may not be supported. To improve the compatibility of TakePhoto, when a specific image selection method is not supported, TakePhoto will automatically switch to another available method for selecting images.

Crop Image
API
TakePhoto supports cropping images, whether the image is taken from the camera or selected from the gallery or files. You just need to call the corresponding method of TakePhoto:  

```java
/**
 * Get image from camera and crop
 * @param outPutUri The path where the cropped image will be saved
 * @param options Crop options
 */
void onPickFromCaptureWithCrop(Uri outPutUri, CropOptions options);
/**
 * Get image from gallery and crop
 * @param outPutUri The path where the cropped image will be saved
 * @param options Crop options
 */
void onPickFromGalleryWithCrop(Uri outPutUri, CropOptions options);
/**
 * Get image from file and crop
 * @param outPutUri The path where the cropped image will be saved
 * @param options Crop options
 */
void onPickFromDocumentsWithCrop(Uri outPutUri, CropOptions options);
/**
 * Multi-image selection and crop
 * @param limit The maximum number of images that can be selected
 * @param options Crop options
 */
void onPickMultipleWithCrop(int limit, CropOptions options);
```   
#### Crop a specific image
Additionally, TakePhoto also supports cropping a specific image:     

```java
/**
 * Crop an image
 * @param imageUri The image to be cropped
 * @param outPutUri The path where the cropped image will be saved
 * @param options Crop options
 */
void onCrop(Uri imageUri, Uri outPutUri, CropOptions options) throws TException;
/**
 * Crop multiple images
 * @param multipleCrop The paths of the images to be cropped and their output paths
 * @param options Crop options
 */
void onCrop(MultipleCrop multipleCrop, CropOptions options) throws TException;
```

#### CropOptions
CropOptions is a configuration class for cropping, through which you can customize settings such as the aspect ratio for cropping, maximum output size, and whether to use the built-in cropping tool of TakePhoto.

**Usage:**  

```java
 CropOptions cropOptions=new CropOptions.Builder().setAspectX(1).setAspectY(1).setWithOwnCrop(true).create();  
 getTakePhoto().onPickFromDocumentsWithCrop(imageUri,cropOptions);  
 getTakePhoto().onCrop(imageUri,outPutUri,cropOptions);  

```

Note:
Due to varying levels of system customization by different Android ROM manufacturers, the system may not have a built-in or third-party cropping tool. To improve the compatibility of TakePhoto, when no cropping tool is available, TakePhoto will automatically switch to using its own built-in cropping tool.

Additionally, TakePhoto 4.0+ supports specifying the use of the built-in gallery. For example: takePhoto.setTakePhotoOptions(new TakePhotoOptions.Builder().setWithOwnGallery(true).create());
For more details, refer to the documentation[Demo](https://github.com/mehroz-dev/TakePhoto/blob/master/simple/src/main/java/com/jph/simple/CustomHelper.java)

Compress Image
You can choose whether to compress the image. You only need to specify whether to enable the compression feature and provide the CompressConfig.  

#### API
```java
 /**
 * Enable image compression
 * @param config The image compression configuration
 * @param showCompressDialog Whether to show the progress dialog during compression
 * @return
 */
 void onEnableCompress(CompressConfig config,boolean showCompressDialog);
```

**Usage:**  

```java
TakePhoto takePhoto=getTakePhoto();
takePhoto.onEnableCompress(compressConfig,true);
takePhoto.onPickFromGallery();
```  
If you enable image compression, `TakePhoto` will use `CompressImage` to compress the image. `CompressImage` currently supports compressing both the size and quality of the image. By default, `CompressImage` enables both size and quality compression.

#### Compress a specific image  
Additionally, you can also compress a specific image:    
**Usage:**  

```java
new CompressImageImpl(compressConfig,result.getImages(), new CompressImage.CompressListener() {
    @Override
    public void onCompressSuccess(ArrayList<TImage> images) {
    }
    @Override
    public void onCompressFailed(ArrayList<TImage> images, String msg) {
    }
}).compress();
```

#### CompressConfig  
`CompressConfig` is a configuration class for image compression. You can use `CompressConfig.Builder` to set the dimensions and quality of the compressed image. If you want to change the compression method, you can make relevant settings using `CompressConfig.Builder`.


**Usage:**   

```java
CompressConfig compressConfig=new CompressConfig.Builder().setMaxSize(50*1024).setMaxPixel(800).create();
```
Specifying Compression Tools
Using TakePhoto Compression Tool:

```
CompressConfig config = new CompressConfig.Builder()
                    .setMaxSize(maxSize)
                    .setMaxPixel(width >= height ? width : height)
                    .create();
takePhoto.onEnableCompress(config, showProgressBar);
```

Using Luban for Compression: 
```
LubanOptions option = new LubanOptions.Builder()
                    .setGear(Luban.CUSTOM_GEAR)
                    .setMaxHeight(height)
                    .setMaxWidth(width)
                    .setMaxSize(maxSize)
                    .create();
CompressConfig config = CompressConfig.ofLuban(option);
takePhoto.onEnableCompress(config, showProgressBar);
```

For more details, refer to the demo:[CustomHelper.java](https://github.com/mehroz-dev/TakePhoto/blob/master/simple/src/main/java/com/jph/simple/CustomHelper.java)


Compatibility
Android 6.0
With the introduction of "Runtime Permissions" in Android 6.0, TakePhoto added automatic permission management. When TakePhoto detects a permission is needed, it will automatically request it, so you don’t need to worry about permission issues.

Android 7.0
In Android N, Android implemented StrictMode, which changed how files are shared between apps. To adapt to these changes and make it easier for users, TakePhoto automatically adjusts based on the phone’s Android version. You can still pass Uri imageUri = Uri.fromFile(file); to TakePhoto without worrying about compatibility issues.

Achieving Higher Compatibility
TakePhoto is written based on the official Android API standards and supports most mainstream ROMs in the market. If you encounter any compatibility issues during use, you can submit Issues.

To adapt to phones that may recycle the Activity during photo capture, TakePhoto performs recovery handling in onSaveInstanceState and onCreate.
To adapt to phones where the screen orientation changes when taking a photo or selecting an image from the gallery, causing the photo to fail, you can add the following configuration in AndroidManifest.xml for the Activity using TakePhoto:
android:configChanges="orientation|keyboardHidden|screenSize"
Example:

```
<activity
    android:name=".MainActivity"
    android:screenOrientation="portrait"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:label="@string/app_name" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```
Contributions
If you encounter any issues while using TakePhoto, you can submit Issues. Contributions to TakePhoto are also welcome, feel free to Fork and Submit Pull Requests.

Update Log

v4.1.0(2018/4/2)
-----------------

1. Upgrade glide to 4.6.1；
2. Upgrade  buildToolsVersion & targetSdkVersion ；
3. rename package name ;

v4.0.3 (2017/1/18)
After successful compression, return the original image path (originalPath) so the user can process the original image themselves.
After successful compression, change the compressed path from path to compressPath.
After successful compression, return the image source type, now categorized as CAMERA or OTHER.
Users can configure CompressConfig.enableReserveRaw(boolean) method, set true to retain the original image, false to delete the original image. This configuration only works if the source type is CAMERA.
Corrected the camera rotation angle feature to be optional.
Final Notes
Code Obfuscation
If you have enabled code obfuscation in your project, you can add the following code to your obfuscation rules file (e.g., proguard-rules.pro):   

```
-keep class com.jph.takephoto.** { *; }
-dontwarn com.jph.takephoto.**

-keep class com.darsh.multipleimageselect.** { *; }
-dontwarn com.darsh.multipleimageselect.**

-keep class com.soundcloud.android.crop.** { *; }
-dontwarn com.soundcloud.android.crop.**

```
