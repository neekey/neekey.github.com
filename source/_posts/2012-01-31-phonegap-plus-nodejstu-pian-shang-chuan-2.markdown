---
layout: post
title: "phoneGap+nodejs图片上传(二)"
date: 2012-01-31 16:50
comments: true
categories: [ nodejs, phoneGap ]
---

上一篇文章已经介绍了客户端（phoneGap）中图片上传的相关技术。下面介绍服务器端（nodejs）对图片上传的处理。

在对服务器端这边进行编码之前，在网上找了几篇关于nodejs处理文件上传的文章，基本上是使用[Connect Form](https://github.com/visionmedia/connect-form)这个模块，但是实际调试的时候发现异常。仔细看了一下这个模块的说明，原来目前已经被废弃，并最终会被移除。对于文件上传的处理，`bodyParser()`目前已经直接支持。

<!--more-->

###上传文件信息获取

因此，对于文件上传的处理，首先需要引入`bodyParser()`:

```javascript
app.configure(function(){
    app.use(express.bodyParser());
});
```

然后通过`req`中的`files`，根据文件的字段名来获取上传的文件信息。

```javascript
app.post( '/upload', function( req, res ){
            
	var image = req.files.image;
    console.log( image );
	console.log( image.path );
	console.log( image.length );
 	console.log( image.filename );
  	console.log( image.mime );
});
```

上面的代码`req.files.image`中的`image`是我在客户端中指定的字段名，然后我们可以通过这个文件信息对象，获取到文件保存到服务器端的路径(`path`),文件的大小(`length`),文件名(`filename`),文件的MIME值(`mime`)。

###图片类型验证

通过上面的信息，我们可以利用`fs`模块，对图片进行重命名等操作。但是在介绍客户端中的上传时，提到过，我们利用phoneGap选取文件的时候，无法获取文件的类型信息（MIME类型甚至没有后缀名）。因此在服务器端，我们不能仅仅凭借客户端发送过来的MIME值来验证文件类型。

解决这个问题最根本的方法是解析二进制的文件头，借此来判断文件的实际类型。根据这个需求，我写了一个简单的模块[gettype](https://github.com/neekey/gettype).

这个模块的API非常简单,给定文件的路径，然后文件类型会在回调函数中作为参数返回。不过因为自身需求有限，暂时只支持常用图片类型`JPEG|PNG|BMP|GIF`的判断。

```javascript
var GetType = require( 'getType' );
var pathToParse = 'images/jpeg.jpg';

GetType.parse( pathToParse, function( err, type ){

    if( err ){
        console.log( 'file format parse error!' );
    }
    else {
        console.log( 'file format is : ' + type );
    }
});
```

完成了上面两部，基本上后端这边主要的技术难点就解决了，接下来还需要完成：

* 结合上面两个技术点做文件的大小和类型验证
* 文件重命名
* 将文件信息写入到数据库
* 想客户端返回结果