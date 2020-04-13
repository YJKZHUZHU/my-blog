##### `Promise`实现

###### `Promise`声明

***

`Promise`是一个构造函数，可以用`new`创建，那就用`class`来创建

- 由于`new Promise((resolve,reject) => {})`所以传入依噶参数（函数），

  [秘籍]: https://promisesaplus.com/	"秘籍"

  里叫他`executor`传入就执行

- `executor`里面有两个参数，一个叫`resolve`,一个叫`reject`

- 由于resolve和reject可执行，所以都是函数，我们用let声明。

```javascript
class MyPromise {
  constructor(executor){
    //成功
    let resolve = (value) => {console.log(value)}
    //失败
    let reject = (error) => {console.log(error)}
    //立即执行
    executor(resolve,reject)
  }
}

new MyPromise((resolve,reject)=> {
  resolve(1), reject(2)
})
```

###### 解决基本状态

***

- `Promise`有三种状态（`state`）:`pending`,`fulfilled`,`rejected`
- `pending`(等待态)为初始态，并可以转化为fulfilled（成功态）和rejected（失败态）
- 成功时，不可转为其他状态，且必须有一个不可改变的值（value）
- 失败时，不可转为其他状态，且必须有一个不可改变的原因（reason）
- `new Promise((resolve, reject)=>{resolve(value)})` resolve为成功，接收参数value，状态改变为fulfilled，不可再次改变
- `new Promise((resolve, reject)=>{reject(reason)})` reject为失败，接收参数reason，状态改变为rejected，不可再次改变
- 若是executor函数报错 直接执行reject()

