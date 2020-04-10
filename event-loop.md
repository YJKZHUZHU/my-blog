##### 浏览器的事件循环机制

###### **`Event loop`是什么**

***

==**event loop是一个执行模型，在不同的地方有不同的实现。浏览器和NodeJS基于不同的技术实现了各自的Event Loop。**==

- 浏览器的Event Loop是在[html5的规范](https://www.w3.org/TR/html5/webappapis.html#event-loops)中明确定义。
- NodeJS的Event Loop是基于libuv实现的。可以参考Node的[官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)以及libuv的[官方文档](http://docs.libuv.org/en/v1.x/design.html)。
- libuv已经对Event Loop做出了实现，而HTML5规范中只是定义了浏览器中Event Loop的模型，具体的实现留给了浏览器厂商。

###### 宏队列和微队列

***

**宏队列，macrotask，也叫tasks。** 一些异步任务的回调会依次进入macro task queue，等待后续被调用，这些异步任务包括：

- **`setTimeout`**
- **`setInterval`**
- setImmediate (Node独有)
- requestAnimationFrame (浏览器独有)
- I/O
- UI rendering (浏览器独有)

**微队列，microtask，也叫jobs。** 另一些异步任务的回调会依次进入micro task queue，等待后续被调用，这些异步任务包括：

- process.nextTick (Node独有)
- Promise
- Object.observe
- MutationObserver

###### **浏览器的`Event loop`**

我们先来看一张图，再看完这篇文章后，请返回来再仔细看一下这张图，相信你会有更深的理解。

![](https://raw.githubusercontent.com/YJKZHUZHU/blog-image-upload/master/img/20200410113004.png)

这张图将浏览器的`Event Loop`完整的描述出来了，下面是执行`Javascript`代码的具体流程

1. 执行全局Script同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
2. 全局Script代码执行完毕后，调用栈Stack会清空；
3. 从微队列microtask queue中取出位于队首的回调任务，放入调用栈Stack中执行，执行完后microtask queue长度减1；
4. 继续取出位于队首的任务，放入调用栈Stack中执行，以此类推，直到直到把microtask queue中的所有任务都执行完毕。**注意，如果在执行microtask的过程中，又产生了microtask，那么会加入到队列的末尾，也会在这个周期被调用执行**；
5. microtask queue中的所有任务都执行完毕，此时microtask queue为空队列，调用栈Stack也为空；
6. 取出宏队列macrotask queue中位于队首的任务，放入Stack中执行；
7. 执行完毕后，调用栈Stack为空；
8. 重复第3-7个步骤；
9. 重复第3-7个步骤；
10. 。。。。

**可以看到，这就是浏览器的事件循环Event Loop**

这里归纳3个重点：

- 宏队列macrotask一次只从队列中取一个任务执行，执行完后就去执行微任务队列中的任务；
- 微任务队列中所有的任务都会被依次取出来执行，直到microtask queue为空；
- 图中没有画UI rendering的节点，因为这个是由浏览器自行判断决定的，但是只要执行UI rendering，它的节点是在执行完所有的microtask之后，下一个macrotask之前，紧跟着执行UI render。

###### 示例

```javascript
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3)
  });
});

new Promise((resolve, reject) => {
  console.log(4)
  resolve(5)
}).then((data) => {
  console.log(data);
})

setTimeout(() => {
  console.log(6);
})

console.log(7);
```

运行结果

```javascript
1
4
7
5
2
3
6
```

分析整个流程

***

1. 执行全局`Script`代码

***

- **step 1**

  ```javascript
  conbsole.log(1)
  ```

  Stack Queue: [console]

  Macrotask Queue: []

  Microtask Queue: []

> 打印结果：
>
> 1

- **step 2**

  ```javascript
  setTimeout(() => {
    // 这个回调函数叫做callback1，setTimeout属于macrotask，所以放到macrotask queue中
    console.log(2);
    Promise.resolve().then(() => {
      console.log(3)
    });
  });
  ```

  Stack Queue: [setTimeout]

  Macrotask Queue: [callback1]

  Microtask Queue: []

> 打印结果
>
> 1

- **step 3**

  ```javascript
  new Promise((resolve, reject) => {
    //注意，这里是同步执行的哦
    console.log(4)
    resolve(5)
  }).then((data) => {
    //这个回调函数叫做callback2,promise属于microtask,所以放到microtask quene 中
    console.log(data);
  })
  ```

  Stack Queue: [promise]

  Macrotask Queue: [callback1]

  Microtask Queue: [callback2]

>打印结果
>
>1
>
>4

- **step 4**

  ```javascript
  setTimeout(() => {
    //这个回调函数叫做callback3，setTimeout属于macrotask，所以放到macrotask queue中
    console.log(6);
  })
  ```

  Stack Queue: [setTimeout]

  Macrotask Queue: [callback1, callback3]

  Microtask Queue: [callback2]

> 打印结果
>
> 1
>
> 4

- **step 5**

  ```javascript
  console.log(7);
  ```

  Stack Queue: [console]

  Macrotask Queue: [callback1, callback3]

  Microtask Queue: [callback2]

> 打印结果
>
> 1
>
> 4
>
> 7

***

2. 好啦，全局Script代码执行完了，进入下一个步骤，从microtask queue中依次取出任务执行，直到microtask queue队列为空。

***

- **step 6**

  ```javascript
  console.log(data); //这里的data是promise决议值5
  ```

  Stack Queue: [callback2]

  Macrotask Queue: [callback1, callback3]

  Microtask Queue: []

> 打印结果
>
> 1
>
> 4
>
> 7
>
> 5

***

3. 这里microtask queue中只有一个任务，执行完后开始从宏任务队列macrotask queue中取位于队首的任务执行

***

- **step 7**

  ```javascript
  console.log(2)
  ```

  Stack Queue: [callback1]

  Macrotask Queue: [callback3]

  Microtask Queue: []

> 打印结果
>
> 1
>
> 4
>
> 7
>
> 5
>
> 2
>
> 但是，执行callback1的时候又遇到了另一个Promise，Promise异步执行完后在microtask queue中又注册了一个callback4回调函数

- **step 8**

  ```javascript
  Promise.resolve().then(() => {
    // 这个回调函数叫做callback4，promise属于microtask，所以放到microtask queue中
    console.log(3)
  });
  ```

  Stack Queue: [promise]

  Macrotask v: [callback3]

  Microtask Queue: [callback4]

> 打印结果
>
> 1
>
> 4
>
> 7
>
> 5
>
> 2

***

4. 取出一个宏任务macrotask执行完毕，然后再去微任务队列microtask queue中依次取出执行

***

- **step 9**

  ```javascript
   console.log(3)
  ```

  Stack Queue: [callback4]

  Macrotask Queue: [callback3]

  Microtask Queue: []

> 打印结果
>
> 1
>
> 4
>
> 7
>
> 5
>
> 2

***

5. 微任务队列全部执行完，再去宏任务队列中取第一个任务执行

***

- **step 10**

  ```javascript
  console.log(6)
  ```

  Stack Queue: [callback3]

  Macrotask Queue: []

  Microtask Queue: []

> 打印结果
>
> 1
>
> 4
>
> 7
>
> 5
>
> 2
>
> 6

***

6. 以上，全部执行完后，Stack Queue为空，Macrotask Queue为空，Micro Queue为空

***

- **step 11**

  Stack Queue: []

  Macrotask Queue: []

  Microtask Queue: []

> 打印结果
>
> 1
>
> 4
>
> 7
>
> 5
>
> 2
>
> 6

​	

