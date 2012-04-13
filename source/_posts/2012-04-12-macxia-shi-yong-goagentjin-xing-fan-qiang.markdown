---
layout: post
title: "Mac下使用goagent进行翻墙"
date: 2012-04-12 16:40
comments: true
categories: [翻墙, python, google, Google Appengine ]
---

![goagent](/images/posts/goagent.png)

这两天花钱买的SSH账号各种不给力，要么连接不上，要么频频掉线。无奈，想起同事**@渐飞**有一次在分享的免费翻墙利器goagent，于是便尝试了下。基本可用，本文简单介绍一下goagent在mac下的使用。

###goagent是什么
goagent是使用Python和Google Appengine SDK编写的网络软件，可以运行在Windows/Mac/Linux/Android/iTouch/iPhone/iPad/webOS/OpenWRT/Maemo上。

<!--more-->

###安装和部署

* 申请[Google Appengine](https://appengine.google.com/start)并创建app。在创建App的时候会让你自定义一个appliction identifier 这个便是下面需要用到的appid

![newapp](/images/posts/goagent-newapp.png)

![appid](/images/posts/goagent-appid.png)


* 进入goagent的[首页](http://code.google.com/p/goagent/)（本身已经提供了非常清楚的教程）
* 下载稳定版的goagent，解压后放在你自己希望放的位置
* 配置：修改`local\proxy.ini`中的`[gae]`下的`appid=你的appid`(多appid请用|隔开)
* 部署：进行server目录，执行 `python uploader.zip` 进行应用的部署，过程中会提示你输入你的google账号和密码，之后会不断检查部署是否成功，成功部署后会提示`Deployment successful`
* 运行goagent：进入local目录，执行`python proxy.py`就可以了。
* 添加证书：添加local/CA.crt和local/certs/下的所有证书

![goagent](/images/posts/goagent-cert.png)

* chrome请安装[SwitchySharp](https://chrome.google.com/webstore/detail/dpplabbmogkhghncfbfdeeokoefdjegm)插件，然后导入这个设置[http://goagent.googlecode.com/files/SwitchyOptions.bak](http://goagent.googlecode.com/files/SwitchyOptions.bak)，firefox请安装AutoProxy

###设置快速启动

这方面我比较笨，一般我都是设置`alias`来达到一个命令启动翻墙的功能。

假设我的goagent在`/users/neekey/goagent/`目录下，那么打开`.bath_profile`

`mate ~/.bath_profile`

添加命令

	#goagent
	alias goagent="python /users/neekey/goagent/local/proxy.py"
	
以后就可以在终端中直接通过`goagent`命令来启动了

###与SSH的比较

与SSH相比，速度上差不多（当然这个和每个人用的SSH服务器有关），不过比SSH肯定更加方便（我每次都是用在终端中输入命令的形式连接ssh，因此每次都要输入一遍密码）。不过goagent的好处还是**绿色，免费!!**，并且相比来说会更加稳定（不过这个要看google了=.=）

试用中，以后有想法会继续更新。