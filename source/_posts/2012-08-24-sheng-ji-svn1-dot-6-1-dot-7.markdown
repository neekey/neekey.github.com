---
layout: post
title: "升级SVN1.6-1.7"
date: 2012-08-24 17:06
comments: true
categories: 
---

SVN这个东西虽然不好用（当然也是因为自身没有重视，但是谁让GIT这么好用呢...!），但是公司暂时用的还是SVN，因此还是不得不每天应对它。

由于一直用`phpstorm`因此SVN的操作都是GUI，非常方便（推荐...!）,但是今天由于有个需求我需要写个命令让SVN自动更新，因此需要在终端中直接使用SVN的命令行工具，但是...关键时刻就给我掉链子了!

<!--more-->

在终端中输入:
```
svn update
```

然后就悲剧了，提示项目的svn是1.7及以上的，我的svn版本太低!表示记得前一阵子刚刚更新过了的。用`svn version`一看，果然是1.6.

但是1.7确实是记得手动更新过的，不甘心，在目录中找，果然在`/usr/local/bin/`中找到了1.7的svn版本，然后在`/usr/bin/`中找到了1.6版本的svn!

系统默认先调用`/usr/bin`中的!

于是定位到原因，估计是升级系统到`moutain lion`的时候，又自动在`/usr/bin`中给我安装了系统自带的`1.6`的svn，于是覆盖了我的1.7的...悲剧!

于是...最简单的方法，就是把1.7的覆盖掉`local/bin`中的1.6版本!覆盖完之后，在终端中查看，果然`version`变成1.7了，于是继续`svn update`，结果接续报错：

```
svn: E170000: Unrecognized URL scheme for http*
```
F******k!

好吧，Google之，得到答案如下：[答案](http://azimbabu.blogspot.com/2008/07/using-svn-and-got-unrecognized-url.html)

没细看...反正SVN要使用`http`类型的仓库需要某块支持，然后报这个错误意味着它可能找不到一个名为`neon`的模块了。因此需要重新编译安装svn，告诉它这个模块在哪里。

因此做法是：

* 下载最新的[neon](http://www.webdav.org/neon)，然后`./configure`, `make`, `make install`安装好
* 配置SVN源码 :
```bash
$  ./configure --with-ssl --with-apr=/usr/local/apache2/bin/apr-config --with-apr-util=/usr/local/apache2/bin/apu-config --with-neon=/usr/local
```
* 编译安装：`make`, `make install`

在配置SVN源码之前，细心的我还是发现了我的路径里面不存在/`usr/local/apache2`这个目录，看了下面的评论，这个应该是在你要配置一台svn服务器时使用，因此像我这种情况，就直接使用
```bash
$  ./configure --with-ssl --with-neon=/usr/local
```
就OK了!




