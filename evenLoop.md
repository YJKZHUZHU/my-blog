##### 浏览器的事件循环机制

###### **`Event loop`是什么**

***

==**event loop是一个执行模型，在不同的地方有不同的实现。浏览器和NodeJS基于不同的技术实现了各自的Event Loop。**==

- 浏览器的Event Loop是在[html5的规范](https://www.w3.org/TR/html5/webappapis.html#event-loops)中明确定义。
- NodeJS的Event Loop是基于libuv实现的。可以参考Node的[官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)以及libuv的[官方文档](http://docs.libuv.org/en/v1.x/design.html)。
- libuv已经对Event Loop做出了实现，而HTML5规范中只是定义了浏览器中Event Loop的模型，具体的实现留给了浏览器厂商。
- 