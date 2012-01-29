---
layout: post
title: "phoneGap+nodejs图片上传(一)"
date: 2012-01-26 17:13
comments: true
categories: [ nodejs, phoneGap ]
---

**继续捣鼓毕设！**

前两天将登陆和注册这些通用功能搞定后，现在需要解决从手机端上传图片到服务器的需求。先来一张目前登陆界面截图吧：

![secondaryTrading_login](/images/posts/st_login.jpg)

手机上传图片的功能主要有两方面:

* 客户端，phoneGap是否提供了接口可以让用户选择图片以及拍照上传。选择了图片后的上传方式又是如何。
* 服务器端，主要是接受客户端的请求，对图片信息进行验证和保存。

<!--more-->

##phoneGap##

*注意，下面的代码实例在android中运行正常，但是不同平台可能会有区别，具体应用中，请仔细阅读API后面对不同平台的Hack!!*

好，回到主题！phoneGap这边主要分成两块内容

###一、图片的获取###

图片的获取主要靠API中的Camera对象

**Camera**

> The camera object provides access to the device's default camera application.

Camera对象提供了方法来调用系统默认的拍照程序。

Camera对象只有一个方法`getPicture`,通过给定options，可以指定返回图片的`base64编码字符串`或者图片的本地URI。

**getPicture**

``` javascript
Camera.getPicture( cameraSuccess, cameraError, [ cameraOptions ] );
```

**cameraOptions**

* quality: 保存的图片质量（个人觉得应该是指拍照的情况），数字，范围为：0-100
* destinationType: 选择返回的图片数据格式。`URI`或者是`base64 encoded string`,这两个值定义如下：

``` javascript
Camera.DestinationType = {
	DATA_URL : 0,                // Return image as base64 encoded string
	FILE_URI : 1                 // Return image file URI
};
```
* sourceType: 设置图片的来源，拍照或者从图库中寻找（但是实际有三种选项，`PHOTOLIBRARY`和`SAVEDPHOTOALBUM`和区别不是很清楚哈～）

``` javascript
Camera.PictureSourceType = {
    PHOTOLIBRARY : 0,
    CAMERA : 1,
    SAVEDPHOTOALBUM : 2
};
```
* allowEdit: 允许在图片被确认选择前进行简单的编辑
* EncodingType: 设置照片的格式（应该是针对拍照的情况）

```
Camera.EncodingType = {
	JPEG : 0,               // Return JPEG encoded image
	PNG : 1                 // Return PNG encoded image
};
```
* targetWidth: 设定图片的宽度
* targetHeight: 设定图片的高度
* MediaType: 设置可以选取的文件类型（可以是只有图片，只有视频，或者都可以），这个选项只有在`sourceType`不是`Camera.PictureSourceType.CAMERA`才起作用

```
Camera.MediaType = { 
    PICTURE: 0,             // allow selection of still pictures only. DEFAULT. Will return format specified via DestinationType
    VIDEO: 1,               // allow selection of video only, WILL ALWAYS RETURN FILE_URI
    ALLMEDIA : 2            // allow selection from all media types
}
```

**代码示例**


``` javascript
navigator.camera.getPicture(onSuccess, onError, {
	quality: 50,
	destinationType: Camera.DestinationType.FILE_URI,
	sourceType: navigator.camera.PictureSourceType.PHOTOLIBRARY
});

function onSuccess( imageURI ) {
	alert( imageURI );
};

function onError( msg ){
	alert( '图片获取失败:' + msg );
}
```

注意下面设置返回为图片的URI，除此之外没有其他参数。图片URI大概是这个样子：
`content://media/external/images/media/4`
我们将这段`URI`设置给`img`元素的`src`属性来实现本地图片，但是除此之外，**貌似无法知道图片的具体格式**，所以后面会涉及到后端nodejs的后端校验。


###二、图片上传###

上面介绍了如何让用户选取图片，下面介绍下文件的上传。

phoneGap中封装了对手机中的文件进行操作的一些常用方法。其中`FileTransfer`对象允许你通过调用其实例对象的`upload`方法将本地文件上传到服务器。

`FileTransfer`使用`HTTP multi-part POST`请求. 支持`HTTP`和`HTTPS`协议. 可以通过传递一个`FileUploadOptions`对象到`upload`方法，来设置上传相关的参数. 

**upload**方法的具体使用直接看代码吧：

```javascript
var options = new FileUploadOptions();
var ft = new FileTransfer();

options.fileKey= 'image';	// 相当与表单中的name字段
options.fileName= '图片名称';
options.mimeType= "image/png";
options.params = {
	field1: 'value1',
	field2: 'value2',
	...
};							// 跟随文件一起发送的自定义额外的字段

ft.upload( imageURI, url, op.success, op.error, options );

function success( fileUploadResult ){
	alert( '文件上传成功' );
}

function error( fileTransferError ){
	alert( '文件上传失败' );
}
```

接下来简单介绍`FileUploadResult`和`FileTransferError`

**FileUploadResult**

Properties:

* bytesSent：总共传送到服务器的字节数
* responseCode：服务器返回的HTTP响应码
* response：服务其返回数据

**fileTransferError**

文件操作相关的方法出现的异常对象都是使用该对象。

Properties:

* code: 预先定义的错误代码.

预定义的代码有以下三种：

* FileTransferError.FILE_NOT_FOUND_ERR 文件未找到
* FileTransferError.INVALID_URL_ERR 非法的URL
* FileTransferError.CONNECTION_ERR 连接错误

**OK！phoneGap这边主要就是上面两个方法，下一篇文章继续介绍nodejs这边的细节。**







