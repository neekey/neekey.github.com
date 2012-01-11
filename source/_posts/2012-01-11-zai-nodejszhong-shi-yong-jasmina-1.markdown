---
layout: post
title: "在nodejs中使用jasmina(一)"
date: 2012-01-11 21:36
comments: true
categories: 
---

寒假在家，终于可以开始好好做毕设！

今天在写后台用户数据操作时，想到每次写类似的模块，在大体完成后总会出现各种Bug，往往调试占用了大量的时间。因此决定这次为每个接口都写一下单元测试。

由于之前听过同事关于[Jasmine][1]的分享，感觉还不错，所以就用它了。

![Jasmine-logo](/images/posts/jasmine_logo.png)

<!-- more -->

###为nodejs添加jasmine模块###

由于后端使用[nodeJS](http://nodejs.org/)开发，因此先用[npm](http://npmjs.org/)安装[Jasmine][1]

    npm install jasmine-node

安装完成后，就只可以直接在终端中使用jasmine-node命令了

###使用Jasmine-node命令###

**Jasmine-node 参数**

``` bash
USAGE: jasmine-node [--color|--noColor] [--verbose] [--coffee] directory

Options:
  --color            - use color coding for output
  --noColor          - do not use color coding for output
  -m, --match REGEXP - load only specs containing "REGEXPspec"
  --verbose          - print extra information per each test run
  --coffee           - load coffee-script which allows execution .coffee files
  --junitreport      - export tests results as junitreport xml format
  --teamcity         - converts all console output to teamcity custom test runner commands. (Normally auto detected.)
  --runWithRequireJs - loads all specs using requirejs instead of node's native require method
  --test-dir         - the absolute root directory path where tests are located
  --nohelpers        - does not load helpers.
  -h, --help         - display this help and exit
```

**使用简单说明**

* 指定目录进行单元测试：
	
```
$ jasmine-node test

```	

注意指定的目录下的包含单元测试代码的脚本文件必须为js或者coffee格式，并且文件名最后必须为**spec**，比如文件：`userSpec.js`，`user.spec.coffee`

* 常用的参数简单说明
	* verbose 默认的结果只显示成功了多少，失败了多少，然后显示失败的信息，指定该参数后，会将所有的信息都输出
	* test-dir 指定目录的绝对路径 


###如何写测试代码###

由于之前没怎么写过单元测试，因此对测试的理解比较肤浅，我的理解比较简单：

*通过某个过程得到一个结果，然后判断是否这个结果是我们预期的。一般这样的测试，其结果应该是可以预知并且可以对结果进行判断*

**最简单的测试代码**

好，现在我们建立一个test文件夹，从最简单的测试代码开始：

``` javascript
describe( '描述一下这个单元测试', function(){

	it( '算数的结果总是比较容易预测', function(){
		expect( 1 + 2 ).toEqual( 3 );
	});
});
```
将这个文件命名为 simpleTestSpec.js，然后使用命令:

``` bash
$ jasmine-node test
```

可以看到结果：

``` bash
描述一下这个单元测试
  算数的结果总是比较容易预测

Finished in 0.008 seconds
1 test, 1 assertion, 0 failures
```

现在我们故意将测试代码改为`expect(1+2).toEqual(0);`，重新运行一下，得到结果：

```
描述一下这个单元测试
  算数的结果总是比较容易预测

Failures:

  1) 算数的结果总是比较容易预测
   Message:
     Expected 3 to equal 0.
   Stacktrace:
     Error: Expected 3 to equal 0.
    at new <anonymous> (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:102:32)
    at [object Object].toEqual (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:1171:29)
    at [object Object].<anonymous> (/users/neekey/Desktop/mhevery-jasmine-node-d3dc963/spec/TestSpec.js:4:19)
    at [object Object].execute (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:1001:15)
    at [object Object].next_ (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:1790:31)
    at [object Object].start (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:1743:8)
    at [object Object].execute (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:2070:14)
    at [object Object].next_ (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:1790:31)
    at [object Object].start (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:1743:8)
    at [object Object].execute (/usr/local/lib/node_modules/jasmine-node/lib/jasmine-node/jasmine-2.0.0.rc1.js:2215:14)

Finished in 0.011 seconds
1 test, 1 assertion, 1 failure
```

上面的describe可以最为一类测试的**群组**，而it则是这些测试中的其中一**项**。而`expect`就是做一次**断言**。

####异步代码测试####

[Jasmine][1]主要提供了三个方法来让我们实现对异步脚本的测试：

* **runs(function)** 官方的说明是：*runs() blocks by themselves simply run as if they were called directly*, 觉得不是很理解，直接看代码吧，当只有一个`runs()`的情况下，下面两端代码的效果一致：

``` javascript
it('should be a test', function () {
  var foo = 0
  foo++;

  expect(foo).toEqual(1);
});
```

``` javascript
it('should be a test', function () {
  runs( function () {
    var foo = 0
    foo++;

    expect(foo).toEqual(1);
  });
});
```

出现多个`runs()`的时候，他们将以串行的方式执行，需要注意的是，在`runs()`指定的函数内部，`this`是被多个`runs()`共享的。
下面这个例子，多个`runs()`串行：

``` javascript
it('should be a test', function () {
  runs( function () {
    this.foo = 0
    this.foo++;
    var bar = 0;
    bar++;

    expect(this.foo).toEqual(1);
    expect(bar).toEqual(1);
  });
  runs( function () {
    this.foo++;
    var bar = 0
    bar++;

    expect(this.foo).toEqual(2);
    expect(bar).toEqual(1);
  });
});
```

* **waits(timeout)** 这个方法和`runs()`一起使用，我们可以通过该方法来指定阻塞多久再执行下一个`runs()`

``` javascript
it('should be a test', function () {
  runs(function () {
    this.foo = 0;
    var that = this;
    setTimeout(function () {
      that.foo++;
    }, 250);
  });

  runs(function () {
    expect(this.foo).toEqual(0);
  });

  waits(500);

  runs(function () {
    expect(this.foo).toEqual(1);
  });
});
```

这段代码在执行完第二个`runs()`后没有直接执行第三个`runs()`,知道`waits()`指定的500毫秒到达后，再执行`runs()`。利用这个方法，我们可以对一些异步回调（这些回调的返回时间可以估计）进行测试。

需要注意的是，如果第三个`runs()`内的代码不用runs来包裹，直接写在外面，**将无法被阻塞执行。**


* **waitsFor(function, optional message, optional timeout)** 在很多情况下，我们无法确切地知道回调的在什么时候返回（比如用户的自定义事件，鼠标点击等），这个时候我们就可以使用该方法。该方法只有在给定的function返回了`true`后才会执行下一个`runs()`，还可以指定最长等待时间，如果在这个时间内还没有返回`true`，则显示`optional message`

``` javascript
describe('waitsFor Test', function() {
  it('after 2sec, it will be true', function () {
    var iWillBeTrue = false;
    
	setTimeout( function(){
		iWillBeTrue = true;
	}, 2000 );

	waitsFor(function() {
      return iWillBeTrue;
    }, "maybe i will never be true", 10000 );

    runs(function () {
      expect( iWillBeTrue ).toEqual( true );
    });
  });
});

```

运行一下：

``` bash
waitsFor Test
  after 2sec, it will be true

Finished in 2.013 seconds
1 test, 1 assertion, 0 failures
```

我们去掉`setTimeout`，那么应该`iWillBeTrue`将永远为`false`，重新运行：

``` bash
waitsFor Test
  after 2sec, it will be true

Failures:

  1) after 2sec, it will be true
   Message:
     timeout: timed out after 10000 msec waiting for maybe i will never be true
   Stacktrace:
     undefined

Finished in 10.921 seconds
1 test, 1 assertion, 1 failure
```


----

总体上来说，[Jasmine][1]感觉还不错，特别是提供的这几个异步测试的方法，基本上能满足一般单元测试的需求。

先介绍到这，更多细节后续会跟进。

[1]:http://pivotal.github.com/jasmine/








