---
layout: post
title: "关于利用HTTP 1.1的Chunked encoding进行前端性能优化的调研"
date: 2012-04-21 15:52
comments: true
categories: 
---

![Http/1.1 Chunked Encoding](/images/posts/httpChunkEncoding.png)
###什么是chunked encoding？

可以翻译为分块编码，是一种HTTP/1.1中定义的服务器向客户端发送数据时，对消息体的编码方式。它是服务器的请求响应Headers中`Transfer-Encoding`字段的值之一。

####Transfer Encoding

该字段表明了对于返回的消息体使用何种编码以保证消息在发送者和接受者之间的安全传输。该字段与`Content-Coding`的区别在于该字段应用于消息体，而不是响应实体对象。

<!--more-->

该字段的的基本格式：

`"Transfer-Encoding" ":" 1#transfer-coding`

注意，许多HTTP/1.0应用无法接受该字段。

####Transfer Coding

`transfer-coding`便是`Transfer-Encoding`字段的值，同时在请求Header字段中也可以作为字段`TE`的值，以表明客户端可以接收的编码。所有的`transfer-coding`都对大小写不敏感。

无论通过`transfer-coding`指定给消息体的编码是什么，在编码集合中，最后一个值必须为`chunked`，且该编码只能出现一次。

`transfer-coding`可用的选项有：`chunked`,`identity`,`gzip`,`compress`以及`deflate`

####Chunked Transfer Coding

分块编码将消息体按照顺序分割成一个块序列。每一个块都由`大小指示器`（size indicator），块数据以及紧跟其后的一个可选的包含实体头域的尾部（trailer）组成。这允许发送端能动态生成内容，并能携带有用的信息，这些信息能让接收者判断消息是否接收完整。

大小指示器是用16 进制数字字符串。块编码（chunked encoding）以任一大小为0的块结束，紧接着是尾部（trailer），尾部以一个空行终止。

可以具体看一个例子：

```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1C
and this is the second one

3
con
8
sequence
0
```

可以通过计算确认每一个chunk的size是否正确：

```
"This is the data in the first chunk\r\n"      (37 chars => hex: 0x25)
"and this is the second one\r\n"               (28 chars => hex: 0x1C)
"con"                                          (3  chars => hex: 0x03)
"sequence"                                     (8  chars => hex: 0x08)
```

###分段对脚本执行的影响

####一个引子

我们来看一段简单的代码

下面为`simple.php`文件的一个片段

```javascript
	<script id="A">
        document.location.hash += '--scriptA';
        setTimeout(function(){

            document.location.hash += '--emitByScriptA';
        }, 0 );
    </script>

    <script id="B">
        document.location.hash += '--scriptB';
    </script>
    
```

执行一下，不出错料，hash值显示为：

`--scriptA--scriptB--emitByScriptA`

现在我们做一下修改，使用php的`flush`方法来进行`chunk`的处理：

```
	<script id="A">
        document.location.hash += '--scriptA';
        setTimeout(function(){

            document.location.hash += '--emitByScriptA';
        }, 0 );
    </script>
    
    <?php 
        flush();
        sleep( 1 );
    ?>

    <script id="B">
        document.location.hash += '--scriptB';
    </script>
    
```

这样的话，内容将被分为两段（要验证这一点，可以利用调试工具查看返回的header中是否有`Transfer-Encoding: chunked`, 并用[fiddler](http://fiddler2.com/fiddler2/)等工具查看返回的raw数据，是否符合上面的`chunk`格式）。运行，hash值显示为：

`--scriptA--emitByScriptA--scriptB`

**emitByScriptA**提前执行了！好像chunk的原因导致上面的代码提前执行了的样子！

要解释上面的现象，我们还需要继续做实验，并深入理解JavaScript的**执行**和浏览器**加载**的关系

####请耐心看完下面的测试

细心的读者可能已经注意到了上面的第二个例子中，在flush后家了一个sleep方法，那我们现在把sleep方法去掉，只保留flush方法，重新执行，你会发现和第一个例子一样：

`--scriptA--scriptB--emitByScriptA`

好吧，那我们把flush方法去掉，只留下sleep方法，看看会发生什么：

`--scriptA--emitByScriptA--scriptB`

只不过这一次由于没有flush，是先观察到明显的页面空白，然后所有脚本执行完毕。

那么看来，似乎chunk对于脚本的执行顺序没有本质上的影响。而执行结果顺序的区别，反而是sleep产生了影响！那么sleep的影响是什么？没错，是不同脚本块**加载完成时间**！

-----


继续看下面的测试！

接下来，我们准备了一个**mass.php**里面包含了大概1M左右的文本数据，然后去掉sleep，在flush后面引入该文件:

```php
flush();
include "mass.php"
```

运行，观察到结果：

`--scriptA--emitByScriptA--scriptB`

发现和sleep的效果一致。

我们去掉flush,只剩下mass.php，现在的测试代码是这个样子：

```
<script id="A">…</script>

<?php include "mass.php" ?>

<script id="B">..</script>
```

运行结果还是一样：

`--scriptA--emitByScriptA--scriptB`

至此我们可以进一步证实了，脚本的执行顺序和chunk无关，和不同的独立的代码块的完成载入的时间有关系。

而从上面的几个例子中均可以看到的是，JavaScript脚本是一边加载一边解析，一边执行的。


**我们继续测试！**

相信会有人对下面的这段代码的执行情况感到好奇：

```javascript
	<script>
        
        output();
    </script>

    <script>
        
        function output(){
            console.log( 'output' );
        }
    </script>
```

结果是JavaScript会报错，提示`output`未定义。

在做这个调研之前，我对于JavaScript脚本可以一边加载一边执行的困惑有很多。其中一点就是像上面的这个例子一样的问题：如果页面中尚未加载的部分定义了一些东西（比如全局变量，方法），而上面已经加载的脚本需要这个东西，那怎么执行？而显然，浏览器已经提供了一种机制，可以不用管下方的JavaScript代码块，而直接执行。

我们可以把每个script标签（不管是页面内脚本，还是外部引入的脚本）看做一个互相独立的闭包，只是相对我们在一个文件中写的多个闭包的关系，这些闭包的执行是以此被抛进JavaScript执行线程队列的，且这些闭包的上下文都是Global（window）

有了上面这个结论，我们再看一个例子，如果在script的中间部分chunk会怎么样？

```php
	<script>

        var beginDate = Date.now();

        <?php flush(); sleep(1) ?>

        document.location.hash += '---scriptSleepfor: ' + ( Date.now() -  beginDate );
    </script>
```

结果是 **0**，说明，虽然js可以一遍加载一般执行，但是js可执行的最小单元是一个script，只有一个script快被完全加载完毕且解析完毕后，其中的代码才会执行。

###总结一下关于js执行的几个结论

* 页面中的script标签是独立解析的执行的，因此一个script标签的解析不依赖其他script标签，换句话说，只要一个完整的解析单元完成，且上一个可以被执行的脚本已经执行完毕，这个单元应该就是可以执行的 ---> 此处想要说明，即使页面下方还有很多脚本（script类型的）尚未加载，已经加载且完整的script也是可以执行的
* 不同的script的执行（假设他们的加载和解析非常连贯）可以理解为顺序且间隔极小（至少小于setTimeout=0的情况）并依次抛入js线程队列中。这个结论来自于，当直接flush之后，setTimeout出的方法并未在下一个script之前执行，但是添加了sleep或者下一个chunk的长度特别大（导致加载时间比较长）的情况下，却会导致setTimeout在下一个script之前执行。可以理解为setTimeout是作为第一个script后第一个抛入线程的js脚本块，因此先执行了。
* JavaScript的执行是一边加载一边解析一边执行的，JavaScript执行的最小单元为一个script，只有该script被完全加载完毕，并解析完成后，才会执行。
* 理解各自独立的script块：可以将每个script块理解为一个个会按照循序执行的闭包，特点在于他们都公用一个上下文（ this -> global -> window ）

####那么chunk是否能对前端性能进行优化？

根据上面对于js执行的一些总结，其实chunk对于js的提前执行或者加载没有太大的影响。唯一可以优化的场景是：

一个动态页面，其在响应请求时的总体计算时间比较长，但是它可以先将部分结果通过chunk的方式提前响应，这样提前响应的部分会预先被浏览器解析其中完整的JavaScript在加载完毕后也将被提早执行。


--------

最后在补充点课外知识：

#### 动态页面一般会自动做分段处理(比如Apache)

* 动态文件（比如PHP），在内容很少的情况下，也没有出现chunked标识，在header中存在content-length字段
* 在内容很多的时候出现了即使没有调用flush方法，也出现了chunked标识，且结尾标识也符合chunked规则（结尾为0以表示结束）
* 在php文件中，使用flush方法，可以看到预期的标识位置
