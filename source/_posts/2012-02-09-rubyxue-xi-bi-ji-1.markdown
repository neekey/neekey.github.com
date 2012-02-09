---
layout: post
title: "ruby学习笔记(一)"
date: 2012-02-09 22:18
comments: true
categories: [ruby]
---
![ruby](/images/posts/ruby_logo.gif)

今天…闲来无事，去看了一下[Ruby][0],因为Octopress是用[Ruby][0]写的，有必要学习一下[Ruby][0]。

直接到官网上去，发现[Ruby][0]非常友好，最下方有中文版本的选择，不过感觉翻译的有点蹩脚，还是直接看英文吧。首页右边有常用的几个入口，比如[下载](http://www.ruby-lang.org/en/downloads/)，[二十分钟学习ruby](http://www.ruby-lang.org/en/documentation/quickstart/)，[在浏览器里尝试Ruby](http://tryruby.org/)(这个页面非常有爱，见下图)

<!--more-->

![ruby](/images/posts/try_ruby_in_browser.png)

很认真的花了20分钟看了[二十分钟学习ruby](http://www.ruby-lang.org/en/documentation/quickstart/)，对[Ruby][0]算是有了一定的认识，总结一下：

###**Interactive Ruby**

翻译成中文应该是可交互的[Ruby][0]，其实是可以让你在终端中执行ruby的程序，在Mac下直接在终端中输入`irb`就行了。然后尽情的尝试吧。

###**控制台信息输出**

就是所谓的`print`之类的方法，在[Ruby][0]中是`puts`：
``` ruby
> puts "hello world"
=> hello world
```

###**基本的简单语法**

* nil: 指的是[Ruby][0]中的空值
* 简单的计算：
``` ruby
> 2+3
=> 5
> 2*3
=> 6
```
* 变量赋值：[Ruby][0]中的变量不需要声明
``` ruby
> a = "hello world"
> puts a
=> hello world
```
* 调用方法: 调用方法和传参的方式比较随意，下面几种都可以。如果方法不需要参数，则括号可加可不加。
``` ruby
> Math.sqrt(9)
=> 3
> Math.sqrt 9
=> 3
```
* **字符串只能用双引号**

###**定义方法**

先看例子：
``` ruby
>def h ( name = "world" )
>puts "hello #{name}"
>end
```
上面的例子定义了一个叫`h`的方法，该方法输出`hello world`。用`def`来表示一个方法定义的开始，`end`来表示定义的结束。其中`hello #{name}`和js中的模板引擎语法类似，其中的name可以调用自己的方法，比如需要让输出首字母大写，可以修改为`hello #{name.capitalize}`

###**类**

####类的定义
先看例子：
``` ruby
class Greeter
	def initialize( name = "world" )
		@name = name
	end
	def say_hi
		puts "Hi #{@name}!"
	end
	def say_bye
		puts "Bye #{@name}!"
	end
end
```
类的定义由`class`关键字来声明，然后在`end`之前可以定义各种方法。上面的例子中`initialize`为构造函数，`@name`为定义的私有属性。注意到类中的方法都可以直接利用`@name`来访问该属性。

####实例化
结合上面的代码和下面的代码，可知`name`无法直接访问，下面的例子由于访问`name`而报错：
``` ruby
> g = Greeter.new( "neekey" )
> g.say_hi()
=> Hi neekey
> g.name
NoMethodError: undefined method `name' for…
```
####判断类实例是否具有某方法
其实`class`定义的类本身已经继承了很多方法，我们可以通过下面的两种方式来查看：
``` ruby
> Greeter.instance_methods
=> ["inspect", "taguri", "tap", "clone", "public_methods", "__send__", "taguri=", "instance_variable_defined?", "equal?", "freeze", "say_hi", "extend", "send", "methods", "hash", "dup", "object_id", "instance_variables", "eql?", "to_yaml", "say_bye", "instance_eval", "id", "singleton_methods", "taint", "frozen?", "instance_variable_get", "to_enum", "instance_of?", "display", "to_a", "h", "to_yaml_style", "type", "instance_exec", "protected_methods", "==", "===", "instance_variable_set", "enum_for", "kind_of?", "respond_to?", "to_yaml_properties", "method", "to_s", "class", "private_methods", "=~", "tainted?", "__id__", "untaint", "nil?", "is_a?"]
```
上面输出了类`Greeter`拥有的全部方法，我们也可以通过给`instance_methods`传递`false`来指定只显示自己定义的方法：
``` ruby
>> Greeter.instance_methods(false)
=> ["say_hi", "say_bye"]
```
我们可以使用`respond_to?`(函数名可以直接用?有木有！)来检查一个实例对象是否具有某个方法：
``` ruby
> g.respond_to?("name")
=> false
> g.respond_to?("say_hi")
=> true
```

####类的动态特性

在类已经定义完成后，我们还可以继续对类进行修改：
``` ruby
class Greeter
	attr_accessor :name
end
```
上面代码对class进行了修改，使得`name`可以被访问，注意上面的代码并未覆盖原有定义，而是**增加**了定义。并且这些修改将马上在新的实例化对象和**已经实例化**的对象身上起作用。
``` ruby
> g.name
=> "neekey"
> g.name = "nic"
> g.name
=> nic"
```
从上面的例子来看，似乎`name`已经从私有变成了公共可被外部访问了（不知道ruby里面是否存在这样的术语）。但是真的是这样么？我们继续看：

``` ruby
> g.respond_to?("name")
=> true
> g.respond_to?("name=")
=> true
```
可以看到，其实通过`attr_accessor`的定义，**不是**将name变成所谓的`公有属性`，而是添加了该属性的`getter`和`setter`，完全可以用下面的方式来写：
``` ruby
> g.name()
=> nic
> g.name=("neekey")
=> neekey
```
另外从构造函数中的`@name=name`一句可以看到，[Ruby][0]的属性是可以动态添加的。

上面就是教程的大体内容。有时间继续学习~

[0]:http://www.ruby-lang.org





