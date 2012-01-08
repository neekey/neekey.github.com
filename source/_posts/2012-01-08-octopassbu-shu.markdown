---
layout: post
title: "Octopass部署"
date: 2012-01-08 20:32
comments: true
categories: 
---

部署可以参考：[Octopress Setup](http://octopress.org/docs/setup/)

###本地部署###

**首先是配置Octopress需要的环境：**

* 由于Octopress需要使用到Git，因此需要先安装[Git](http://git-scm.com/)
* ruby 1.9.2 ( 可以使用RVM或者rbenv，他们都是ruby的环境管理软件 )

**安装RVM**

用以下命令安装：

`bash < <(curl -s https://rvm.beginrescueend.com/install/rvm)``

安装完成后，需要修改用户目录下的.bash_profile文件，将RVM设置为shell的一个function:

    echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # Load RVM function' >> ~/.bash_profile
    source ~/.bash_profile

    # If using Zsh do this instead
    echo '[[ -s $HOME/.rvm/scripts/rvm ]] && source $HOME/.rvm/scripts/rvm' >> ~/.zshrc
    source ~/.zshrc

由于我自己对shell命令不是很熟悉，所以简单的查了一下资料。上面的命令中：

* echo `echo 'text' >> targetfile` 将text添加到目标文件尾
* source `source fileHasCodeToRun` 执行指定文件中的脚本

<!-- more -->

**安装ruby 1.9.2**

用安装好的rvm来安装ruby 和 [rubygems](http://rubygems.org/)(ruby的包管理器)

    rvm install 1.9.2 && rvm use 1.9.2
    rvm rubygems latest

**配置Octopass**

首先将Octopass的源码clone到本地

    git clone git://github.com/imathis/octopress.git octopress
    cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
    ruby --version  # Should report Ruby 1.9.2

注意此处一定要确保`ruby --version`是1.9.2，否则后面的步骤会出错。

进入Octopass根目录后，安装依赖包：

    gem install bundler # 应该是一个用来管理依赖的组件(=.=)
    rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
    bundle install

安装完成后，再安装Octopass默认的主题：

    rake install

至此，本地的Octopass就差不多部署完毕了。

---

###部署Octopass到Github Page###

可以直接参考官方的说明：[Github Page](http://pages.github.com)

简单的来说，就是：

* 建立一个repository，命名：reponame.github.com
* 在该 repo 根目录下放置一个index.html
* 通过reponame.github.com来访问

非常简单吧！当然Github Page也支持[Custom Domain](//http://pages.github.com/#custom_domains)

建立好你自己的Github Page后，回到Octopass目录，使用一下命令：
    
    rake setup_github_page

这个命令将：

* 让你输入你的Github Page的repo的url
* Rename the remote pointing to imathis/octopress from ‘origin’ to ‘octopress’(不是很懂…这是原文)
* 将你的Github Page的repo的url作为默认的origin remote

    其实在这里，你在github上的repo对应的本地目录是Octopass/_deploy目录，这个目录下是编译后的静态文件。
* 从master分支转换到source分支

    我自己部署的时候发现没有这个变化。而且所谓的source，我自己认为应该是整个Octopass文件夹（里面的_deploy文件夹下的内容倒是不重要，因为每次`rake generate`都能更新）。我的做法是，`git init`整个Octopass文件夹，然后`add .`把所有文件都以*source*分支提交。

* 配置blog的url指向repo
* 为_deploy目录设置master分支，用户部署

总之，这个命令的作用我自己还是有点**confused**

然后就是生成**_deploy/**目录下的文件，并部署到**github**上:

    rake generate
    rake deploy

那么现在的repo各分支的状态就是：

* master： 对应_deploy目录下的文件
* source： 对应Octopass目录下的所有文件（但是不包括_deploy）

这样我们就可以用source分支来编辑blog，使用master来发布!

**Have Fun！**

---

暂时准备先用这个github的域名，过一阵子觉得真心不错的时候，再考虑直接将我的[neekey.net](http://neekey.net/blog)指向这边好了。
