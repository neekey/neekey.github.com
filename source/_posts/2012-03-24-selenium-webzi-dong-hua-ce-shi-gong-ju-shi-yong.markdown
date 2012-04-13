---
layout: post
title: "Selenium - Web自动化测试工具 试用"
date: 2012-03-24 19:33
comments: true
categories: [ selenium， webdriver， 测试 ]
---

![Selenium](/images/posts/selenium-logo.png)

##什么是Selenium

Selenium是一个web自动化测试工具集。它提供了多种工具来辅助自动化测试的进行。整个完整的套件提供了web应用测试需要的各种部件，并提供了非常灵活和丰富的配置，允许进行本地UI测试以及对期望值与应用的实际表现进行比较。

Selenim最大的优势是它能支持几乎所有的浏览器平台（包括移动web）

<!--more-->

##Selenium简史

Selenium最早在2004年，由ThoughtWorks公司的一位名为Jason Huggins的员工开发的。后来他开发的这个部分成为了Selenium的核心，最终成为Selenium Remote Control（RC）和Selenium IDE的基础。由于可以用可选的语言来对浏览器进行控制，Selenium在当时成为一个巨大的突破。

尽管如此，Selenium还是有很大的缺点。由于它的自动化测试引擎是基于Javascript，而浏览器在安全性方面对Javascript有很多限制，因此很多行为无法在测试中被执行。

在2006年，一位来自Google的名为Simon Stewart的大胆的工程师开启了一个名为WebDriver的项目。Google在很长一段时间曾是Selenium的大客户之一，但是测试人员不得不上被上述的安全性所限制。因此Simon希望能开发出一个工具，可以直接和浏览器进行“对话”，并可以直接使用浏览器和操作系统的“本地”接口。这样就避免了Javascript在沙箱中执行所带来的各种限制。

2008年，除了北京2008奥运会和金融危机外，或许最重要的事情就是Selenium和WebDriver的合并了。

##Selenium测试简单原理

以下仅为个人理解，以后会更新

一个最简单的Selenium工具，应该有以下几部分构成：
* Selenium Server： 用来解析测试脚本，并和webDriver进行通信
* WebDriver： Driver顾名思义是一个驱动，或者说就是一层接口的封装。WebDriver作为Server和browser的连接人。它接受server传来的命令，然后控制浏览器的行为。同时将浏览器中的信息反馈到server中。WebDriver一般会以浏览器的插件形式存在。每个浏览器都会对应一个自己的webDriver。
* 测试脚本： 测试行为的具体编码。Selenium支持很多中语言，比如Java，PHP，Ruby等等，出了官方支持外，也有很多第三方开发人员编写的实现。本文中的实践将使用github上的nodeJS实现（[webDriverJS](https://github.com/Camme/webdriverjs)）

##部署Selenium

###Server
从官网上下载server的jar文件 [download](http://seleniumhq.org/download/)

直接运行该jar文件，变开启了server

`java -jar selenium-server-standalone-2.20.0.jar`

###WebDriver
上面的Selenium简史中介绍过，webDriver和Selenium在08年已经合并，因此大部分浏览器的WebDriver都是由Selenium来维护，因此不需要下载和安装额外的webDriver，一般server那边的运行的时候会直接安装或者附带了。

不过Chrome的webDriver：ChromeDriver是由Google团队自己维护，因此如果需要对chrome进行调试，需要下载ChromeDriver，并为server指定ChromeDriver的路径：

`java -jar selenium-server-standalone-2.20.0.jar -Dwebdriver.chrome.path=PATH_TO_CHROME_DRIVER`

###测试脚本
使用一种你喜欢的语言来编写测试用例，然后执行它！

这边要说明一下测试脚本和Server之间的关系。

实际上测试脚本执行后和Server是一种通信关系，而它们通过Restful的方式进行通信。在开启server后，我们可以看到控制台中有一句这样的说明：

`228 [main] INFO org.openqa.selenium.server.SeleniumServer - RemoteWebDriver instances should connect to: http://127.94.0.1:4444/wd/hub`

说明我们在测试脚本中，要做的第一件事情就是产生一个RemoteWebDriver的实例，然后通过这个实例对象和Server进行通信。并且利用Restful的方式进行通信。为什么说是用Restful的方式，下文会做说明。

##编写我们的测试脚本

本文使用nodeJS进行测试脚本的编写，使用了第三方的接口实现[webDriverJS](https://github.com/Camme/webdriverjs).

直接用npm安装

`npm webdriverjs`

然后新建一个js文件，输入：

```javascript
var webdriverjs = require("webdriverjs");
var client = webdriverjs.remote({
    desiredCapabilities:{
        browserName:"chrome"
    }});
	
client
    .init()
    .url("http://www.google.com")
    .getTitle(function(t){ console.log(t)})
    .end(); 
```   

然后用node执行这个文件，可以看到Chrome自动打开，然后打开Google主页，打开后一下子就关掉了。查看终端我们可以看到下面的结果:

![selenium-code](/images/posts/selenium-code.png)

倒数第二行是我们在代码中log出来的页面的title。

回到刚刚的话题，我们看看为什么说是用Restful方式通信。

前面三行，是在向Server申请会话（session）。webDriver 对象发起了一个`/wd/hub/session`资源请求，并附上了会话的配置参数，然后server返回会话的`id` 


```
[19:13:14]:  COMMAND	POST 	 "/wd/hub/session"
[19:13:14]:  DATA		 {"desiredCapabilities":{"browserName":"chrome","version":"","javascriptEnabled":true,"platform":"ANY"},"sessionId":null}
[19:13:15]:  SET SESSION ID  	 "1332585814526"
```

接下来请求打开指定的url,注意到请求的`uri`中包含了返回的会话`id`：`/wd/hub/session/1332585814526/url`

```
[19:13:15]:  COMMAND	POST 	 "/wd/hub/session/1332585814526/url"
[19:13:15]:  DATA		 {"url":"http://www.google.com"}
```

然后请求页面的title值，这是一个`GET`方式的请求，因此有`RESULT`

```
[19:13:20]:  COMMAND	GET 	 "/wd/hub/session/1332585814526/title"
[19:13:20]:  RESULT	 	 "Google"
```
最后，关闭页面，也就是`end()`，实例对象想Server发起了一个`DELETE`请求，删除对话。

那么说句题外话，其实对于Selenium的测试语言的实现本质上是**对面向开发人员的接口到Selenium的Restful接口的封装**。

##Selenium的优势

* 兼容性 

几乎支持所有浏览器（Selenium2 目前好像还不支持Sarari），包括IE6以及移动浏览器（Android和iOS）

* 没有安全性以及沙盒限制 

之前有用过`jasmine`来进行前端代码测试，但是由于Javascript作为一段页面脚本在页面上执行，因此有很多限制，对于很多测试条件无法实现。比如无法跨域，那么父页面的测试代码要去对子页面的相关情况进行测试就会有很多问题。另外也无法跨页面执行代码，因为一旦页面刷新，所有代码都被刷新了。

由于Selenium2使用了webDriver，因此其脚本的执行是不受安全性沙盒限制的，因此上述的问题都不存在。

* 可以用多种语言编写测试用例

使用jasmine对前端页面进行测试，只能使用javascript，而selenium支持多种语言（JAVA，RUBY，PHP以及各种第三方的实现），因此其受众更广。

##最后附上自己制作的演示视频（勿喷呵呵）

<embed src='http://player.youku.com/player.php/sid/XMzcwNjQwMzI0/v.swf' quality='high' width='480' height='400' align='middle' allowScriptAccess='sameDomain' type='application/x-shockwave-flash'></embed>
