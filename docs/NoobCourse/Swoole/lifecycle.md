<head>
     <title>EasySwoole 入门教程|swoole 入门教程|swoole简介|swoole生命周期</title>
     <meta name="keywords" content="EasySwoole 入门教程|swoole 入门教程|swoole简介|swoole生命周期"/>
     <meta name="description" content="EasySwoole 入门教程|swoole 入门教程|swoole简介|swoole生命周期"/>
</head>
---<head>---
## Swoole的生命周期
------

#### php-fpm中的生命周期
------
传统基于php-fpm的web开发通常淡化了全局期变量的影响，这对于开发者而言，显然是降低了许多上手难度的；但也导致了许多新人对全局期变量会产生的影响没有深刻理解。

我们知道，php-fpm收到请求后会分配一个work进程去处理这条请求，而work会去读取并执行.php文件(在通常情基于框架的开发中，这个.php文件可能是index.php)。也就是说在传统模式中，每个请求都是独立在自己的进程中执行的，因为进程是隔离的而php-fpm又是同步阻塞的，所以我们可以很好的清楚和了解是谁在什么时候创建了变量、修改了变量、销毁了变量。

##### 简单举个例子
可可酱是商店的一名售货员，这个店只有他一个人。当客户来了之后需要购买一瓶可乐，可可酱检查了货架确认有可乐，随后告诉客户这瓶可乐价格是￥3.5元，客户付钱给可可酱，可可酱收到钱后把可乐交给了客户。

后来发现客人太多，只有可可酱一个人的时候后面的客户需要排队很久，于是老板决定再雇一名售货员，于是加入了小明。

还是上面的场景，但是由于小明的加入，小明和可可酱同时接待了2名客户，可可酱检查了货架确认还有一箱可乐的时候，和客人沟通可乐的价格时，小明的客户需要购买一箱可乐，于是小明就取走了一箱可乐，当可可酱收了钱准备拿可乐的时候，发现没有可乐了，然后被客户打了一顿。

在上面的场景中，就是一个变量被修改而导致后续逻辑混乱的场景，在传统的fpm开发中，往往只会在访问数据库的时候出现这种场景。但是如果假设一个fpm进程可以同时处理多条请求的时候，你如果将用户信息存放在全局变量中，那么你就无法再可靠的判断当前用户是谁了。

# swoole_server中对象的4层生命周期
-------
以下内容摘自[swoole文档](https://wiki.swoole.com/wiki/page/354.html)

开发swoole程序与普通LAMP下编程有本质区别。在传统的Web编程中，PHP程序员只需要关注request到达，request结束即可。而在swoole程序中程序员可以操控更大范围，变量/对象可以有四种生存周期。


::: warning 
 变量、对象、资源、require/include的文件等下面统称为对象
:::


程序全局期
-----
在`swoole_server->start`之前就创建好的对象，我们称之为程序全局生命周期。这些变量在程序启动后就会一直存在，直到整个程序结束运行才会销毁。

有一些服务器程序可能会连续运行数月甚至数年才会关闭/重启，那么程序全局期的对象在这段时间持续驻留在内存中的。程序全局对象所占用的内存是`Worker`进程间共享的，不会额外占用内存。

这部分内存会在写时分离（`COW`），在`Worker`进程内对这些对象进行写操作时，会自动从共享内存中分离，变为**进程全局**对象。


::: warning 
 程序全局期`include`/`require`的代码，必须在整个程序`shutdown`时才会释放，`reload`无效
:::


进程全局期
-----
swoole拥有进程生命周期控制的机制，一个`Worker`子进程处理的请求数超过max_request配置后，就会自动销毁。`Worker`进程启动后创建的对象（onWorkerStart中创建的对象），在这个子进程存活周期之内，是常驻内存的。onConnect/onReceive/onClose 中都可以去访问它。


::: warning 
进程全局对象所占用的内存是在当前子进程内存堆的，并非共享内存。对此对象的修改仅在当前`Worker`进程中有效   
:::


::: warning 
 进程期include/require的文件，在`reload`后就会重新加载  
:::

会话期
-----
`onConnect`到`onClose`是一次TCP的会话周期，http keep-alive时，一个连接可能会有多个request。
http是无状态的，一个用户可能也不止一个连接，可以通过创建一个session来关联同一个用户的不同请求。

请求期
----
请求期就是指一个完整的请求发来，也就是`onReceive`收到请求开始处理，直到返回结果发送`response`。这个周期所创建的对象，会在请求完成后销毁。

swoole中请求期对象与普通PHP程序中的对象就是一样的。请求到来时创建，请求结束后销毁。

### 总结
-------
在Swoole中，一个work进程处理完请求后并不会销毁(甚至可能同时处理多个请求)，所以务必要明确你创建的变量的生命周期，以防止出现逻辑上的问题。