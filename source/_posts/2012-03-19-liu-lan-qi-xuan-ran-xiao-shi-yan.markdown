---
layout: post
title: "浏览器渲染小实验"
date: 2012-03-19 10:47
comments: true
categories: 
---

今天做了一个小实验，简单地研究下浏览器对于html的渲染

``` javascript
<div id="div">
	<div id="childDiv">
		<a id="a" href="#">
			<script>
				alert( document.getElementById( 'div' ).innerHTML );
			</script>
		</a>
	</div>
	<script>
		alert( document.getElementById( 'div' ).innerHTML );
		var a = 'test';
	</script>
	<p>
		<script>
			alert( document.getElementById( 'div' ).innerHTML );
		</script>
	</p>
</div>
```

我想要看一下当脚本在html还未闭合的情况下执行，浏览器会如何处理。下面是运行结果截图：

<!--more-->

第一个结果: 只显示了`childDiv`部分

![运行结果1](/images/posts/browser_render1.png)

第二个结果: 比原来多了自己这段`script`

![运行结果1](/images/posts/browser_render2.png)

第三个结果: 终于完全显示出来了

![运行结果1](/images/posts/browser_render3.png)

上面的结果表明了以下几点：

* 浏览器会自动闭合html结构。虽然每个结果中，html的结构输出都是完整的，但是并不等于在`alert`时html都被解析完。比如第一个`script`中，其实`</a>`，`</div>`以及`childDiv`下方的html都还未被解析到，但是由于上面已经有了`<div id="childDiv">`和`<a id="a" href="#">`，因此浏览器自动闭合了。
* 脚本阻塞了渲染
* 脚本部分是真个载入后解析完毕再执行。从第二个`script`中可以看到，虽然在执行`alert`，但是下面的`var a = ..`已经被解析完毕了。

