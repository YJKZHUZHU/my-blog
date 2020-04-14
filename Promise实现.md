##### `Promise`实现

###### `Promise`声明

***

`Promise`是一个构造函数，可以用`new`创建，那就用`class`来创建

- 由于`new Promise((resolve,reject) => {})`所以传入依噶参数（函数），[秘籍](https://promisesaplus.com/) 里叫他`executor`传入就执行
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

```javascript
class MyPromise {
  constructor(executor){
    //初始化状态为等待状态
    this.state = 'pending'
    //成功的值
    this.value = null
    //失败的原因
    this.error = null
    //成功
    let resolve = (value) => {
      if(this.state === 'pending'){
        this.state = 'fulfilled'
        //存储成功的值
        this.value = value
      }
    }
    //失败
    let reject = (error) => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        //存储成功的值
        this.error = error
      }
    }
    //如果executor执行报错，执行rejected
    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
    
  }
}
```

###### `then`方法实现

***

==Promise有一个叫做then的方法，里面有两个参数：onFulfilled,onRejected,成功有成功的值，失败有失败的原因==

- 当状态state为fulfilled，则执行onFulfilled，传入this.value。当状态state为rejected，则执行onRejected，传入this.reason
- onFulfilled,onRejected如果他们是函数，则必须分别在fulfilled，rejected后被调用，value或reason依次作为他们的第一个参数

```javascript
class MyPromise {
  constructor(executor){
    //初始化状态为等待状态
    this.state = 'pending'
    //成功的值
    this.value = null
    //失败的原因
    this.error = null
    //成功
    let resolve = (value) => {
      if(this.state === 'pending'){
        this.state = 'fulfilled'
        //存储成功的值
        this.value = value
      }
    }
    //失败
    let reject = (error) => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        //存储成功的值
        this.error = error
      }
    }
    //如果executor执行报错，执行rejected
    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
    
  }

  then(onFulfilled,onRejected) {
    //状态为fulfilled，执行该函数传入成功的值
    if(this.state === 'fulfilled'){
      onFulfilled(this.value)
    }
     //状态为rejected，执行该函数传入失败的原因
    if(this.state === 'rejected'){
      onRejected(this.error)
    }
  }
}

let aa = 1

const p = new MyPromise((resolve,reject)=> {
  if(aa === 1){
    return resolve(aa)
  }
  reject(aa)
})

p.then((res)=> {
  console.log(res) //1
},(err) => {
    console.log(err)
})

```

对于`setTimeout`和异步请求数据还是没办法使用的，继续改

###### 解决异步实现

***

**现在基本可以实现简单的同步代码，但是当resolve在setTomeout内执行，then时state还是pending等待状态 我们就需要在then调用的时候，将成功和失败存到各自的数组，一旦reject或者resolve，就调用它们**

==加异步操作直接报错==

```javascript
new MyPromise((resolve,reject)=> {
  if(aa === 1){
    setTimeout(() => {
      return resolve(aa)
    },1000)
  }
  reject(aa)
}).then((res) => {
  console.log(res)//TypeError: onRejected is not a function
},(err)=> {
    console.log(err)
})
```

类似于发布订阅，先将then里面的两个函数储存起来，由于一个promise可以有多个then，所以存在同一个数组内。

```javascript
// 多个then的情况
let p = new Promise()
p.then()
p.then()
```

成功或者失败时，forEach调用他们

```javascript
class MyPromise {
  constructor(executor){
    //初始化状态为等待状态
    this.state = 'pending'
    //成功的值
    this.value = null
    //失败的原因
    this.error = null
    //成功存放的数组
    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []
    //成功
    let resolve = (value) => {
      if(this.state === 'pending'){
        this.state = 'fulfilled'
        //存储成功的值
        this.value = value
        //一旦resolve执行，调用成功数组的函数
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    //失败
    let reject = (error) => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        //存储成功的值
        this.error = error
        //一旦reject执行，调用失败数组的函数
        this.onRejectedCallbacks.forEach(fn => fn())
       
      }
    }
    //如果executor执行报错，执行rejected
    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
    
  }

  then(onFulfilled,onRejected) {
    console.log(onFulfilled,onRejected)
    //状态为fulfilled，执行该函数传入成功的值
    if(this.state === 'fulfilled'){
      onFulfilled(this.value)
    }
     //状态为rejected，执行该函数传入失败的原因
    if(this.state === 'rejected'){
      onRejected(this.error)
    }
    //当状态为pending时
    if(this.state === 'pending'){
      this.onResolvedCallbacks.push(() => onFulfilled(this.value))
      this.onRejectedCallbacks.push(() => onRejected(this.error))
    }
  }
}


const p = new MyPromise((resolve,reject)=> {
  const randomNumber = parseInt(Math.random(0, 10) * 10)
  setTimeout(() => {
    if (randomNumber <5){
      resolve(randomNumber)
    }
    reject(randomNumber)
  }, 100)
})

p.then((res)=> {
  console.log(res) 
},(err)=> {
    console.log(err)
})
```

###### **解决链式调用问题**

**我门常常用到`new Promise().then().then()`,这就是链式调用，用来解决回调地狱**

1. 为了达成链式，我们默认在第一个then里返回一个promise。[秘籍](https://promisesaplus.com/)规定了一种方法，就是在then里面返回一个新的promise,称为promise2：`promise2 = new Promise((resolve, reject)=>{})`
   - 将这个promise2返回的值传递到下一个then中
   - 如果返回一个普通的值，则将普通的值传递给下一个then中
2. 当我们在第一个then中`return`了一个参数（参数未知，需判断）。这个return出来的新的promise就是onFulfilled()或onRejected()的值
   - 首先，要看x是不是promise。
   - 如果是promise，则取它的结果，作为新的promise2成功的结果
   - 如果是普通值，直接作为promise2成功的结果
   - 所以要比较x和promise2
   - resolvePromise的参数有promise2（默认返回的promise）、x（我们自己`return`的对象）、resolve、reject
   - resolve和reject是promise2的

```javascript
class MyPromise {
  constructor(executor){
    //初始化状态为等待状态
    this.state = 'pending'
    //成功的值
    this.value = null
    //失败的原因
    this.error = null
    //成功存放的数组
    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []
    //成功
    let resolve = (value) => {
      if(this.state === 'pending'){
        this.state = 'fulfilled'
        //存储成功的值
        this.value = value
        //一旦resolve执行，调用成功数组的函数
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    //失败
    let reject = (error) => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        //存储成功的值
        this.error = error
        //一旦reject执行，调用失败数组的函数
        this.onRejectedCallbacks.forEach(fn => fn())
       
      }
    }
    //如果executor执行报错，执行rejected
    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
    
  }

  then(onFulfilled,onRejected) {
    let promise2 = new MyPromise((resolve,reject)=> {
      //状态为fulfilled，执行该函数传入成功的值
      if (this.state === 'fulfilled') {
        let x = onFulfilled(this.value)
        resolvePromise(promise2,x,resolve,reject)
      }
      //状态为rejected，执行该函数传入失败的原因
      if (this.state === 'rejected') {
        let x = onRejected(this.error)
        resolvePromise(promise2, x, resolve, reject)
      }
      //当状态为pending时
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => {
          let x = onFulfilled(this.value)
          resolvePromise(promise2, x, resolve, reject)
        })
        this.onRejectedCallbacks.push(() => {
          let x = onRejected(this.error)
          resolvePromise(promise2, x, resolve, reject)
        })
      }
    })

    return promise2
   
  }
  
  resolvePromise(promise2, x, resolve, reject){

  }
}
```

###### 完成**resolvePromise**

***

[秘籍](https://promisesaplus.com/)规定了一段代码，让不同的promise代码互相套用，叫做resolvePromise

- 如果 x === promise2，则是会造成循环引用，自己等待自己完成，则报“循环引用”错误

  ```javascript
  const p = new MyPromise((resolve,reject)=> {
    resolve(111)
  })
  
  const p2 = p.then(res => {
    console.log(res)
    return p2
  },err => {
    console.log(err)
  })
  
  p2.then(res => {
    console.log(res)
  },err => {
      console.log(err)//Error: 循环引用,一直是pending状态
  })
  
  ```

- 判断`x`

  - **Otherwise, if x is an object or function,Let then be x.then**

  - `x`不能是`null`

  - x 是普通值 直接resolve(x)

  - x 是对象或者函数（包括promise），`let then = x.then` 2、当x是对象或者函数（默认promise）

  - 声明了then

  - 如果取then报错，则走reject()

  - 如果then是个函数，则用call执行then，第一个参数是this，后面是成功的回调和失败的回调

  - 如果成功的回调还是pormise，就递归继续解析 3、成功和失败只能调用一个 所以设定一个called来防止多次调用

    ```javascript
    function resolvePromise(promise2, x, resolve, reject) {
      //循环应用报错
      if ( promise2 === x) {
        return reject(new Error('循环引用'))
      }
      // 防止多次调用
      let called;
      // x不是null 且x是对象或者函数
      if (x != null && (typeof x === 'object' || typeof x === 'function')) {
        try {
          // A+规定，声明then = x的then方法
          let then = x.then;
          // 如果then是函数，就默认是promise了
          if (typeof then === 'function') {
            // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
            then.call(x, y => {
              // 成功和失败只能调用一个
              if (called) return;
              called = true;
              // resolve的结果依旧是promise 那就继续解析
              resolvePromise(promise2, y, resolve, reject);
            }, err => {
              // 成功和失败只能调用一个
              if (called) return;
              called = true;
              reject(err);// 失败了就失败了
            })
          } else {
            resolve(x); // 直接成功即可
          }
        } catch (e) {
          // 也属于失败
          if (called) return;
          called = true;
          // 取then出错了那就不要在继续执行了
          reject(e);
        }
      } else {
        resolve(x);
      }
    }
    ```

  ###### 解决其他问题

  1. [秘籍](https://promisesaplus.com/)规定onFulfilled,onRejected都是可选参数，如果他们不是函数，必须被忽略
     - onFulfilled返回一个普通的值，成功时直接等于 `value => value`
     - onRejected返回一个普通的值，失败时如果直接等于 value => value，则会跑到下一个then中的onFulfilled中，所以直接扔出一个错误`reason => throw err`
  2. [秘籍](https://promisesaplus.com/)规定onFulfilled或onRejected不能同步被调用，必须异步调用。我们就用setTimeout解决异步问题
     - 如果onFulfilled或onRejected报错，则直接返回reject()

  ```javascript
   then(onFulfilled,onRejected) {
      // onFulfilled如果不是函数，就忽略onFulfilled，直接返回value
      onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
      // onRejected如果不是函数，就忽略onRejected，直接扔出错误
      onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
      let promise2 = new MyPromise((resolve,reject)=> {
        //状态为fulfilled，执行该函数传入成功的值
        if (this.state === 'fulfilled') {
          // let x = onFulfilled(this.value)
          // resolvePromise(promise2, x, resolve, reject)
          // 异步
  
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value)
              resolvePromise(promise2, x, resolve, reject)
            } catch (error) {
              reject(error)
            }
          },0)
        }
        //状态为rejected，执行该函数传入失败的原因
        if (this.state === 'rejected') {
          // 异步
          setTimeout(() => {
            try {
              let x = onRejected(this.error)
              resolvePromise(promise2, x, resolve, reject)
            } catch (error) {
              reject(error)
            }
          }, 0)
         
        }
        //当状态为pending时
        if (this.state === 'pending') {
          this.onResolvedCallbacks.push(() => {
            // 异步
            setTimeout(() => {
             try {
               let x = onFulfilled(this.value)
               resolvePromise(promise2, x, resolve, reject)
             } catch (error) {
               reject(error)
             }
            },0)
            
          })
          this.onRejectedCallbacks.push(() => {
            // 异步
            setTimeout(() => {
              try {
                let x = onRejected(this.error)
                resolvePromise(promise2, x, resolve, reject)
              } catch (error) {
                reject(error)
              }
            },0)
            
          })
        }
      })
      return promise2
    }
  ```

###### 完整代码

```javascript
function resolvePromise(promise2, x, resolve, reject) {
  //循环应用报错
  if ( promise2 === x) {//为了防止回调地狱的形成，需判断promise2和x的关系，如果相等，就出现了回调地狱，此时抛出一个错误
    return reject(new Error('循环引用'))
  }
  // 防止多次调用，promise即可以成功又可以失败，那么这里我们就要判断一下，不能让两次调用都执行，只调用第一个被调用的
  let called;
  // x不是null 且x是对象或者函数
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      // A+规定，声明then = x的then方法
      let then = x.then;//每个promise都有一个then方法，用变量存起来
      // 如果then是函数，就默认是promise了
      if (typeof then === 'function') {
        // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
        then.call(x, y => {//让then方法执行，如果是一个函数，判断执行结果，成功就resolve,失败就直接reject,因为当前then的this并不是指向x,用call改变this指向，让then可以访问x上的实例方法和属性
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          // resolve的结果依旧是promise 那就继续解析，递归执行
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          reject(err);// 失败了就失败了
        })
      } else {
        resolve(x); // 普通值，直接resolve就好
      }
    } catch (e) {
      // 也属于失败
      if (called) return;
      called = true;
      // 取then出错了那就不要在继续执行了
      reject(e);
    }
  } else {
    resolve(x);//如果当前的x值既不是函数也不是object那就说明就是普通类型的值，普通值就进入下一次成功的方法
  }
}
class MyPromise {
  constructor(executor){
    //初始化状态为等待状态
    this.state = 'pending'
    //成功的值
    this.value = null
    //失败的原因
    this.error = null
    //成功存放的数组
    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []
    //成功
    let resolve = (value) => {
      if(this.state === 'pending'){
        this.state = 'fulfilled'
        //存储成功的值
        this.value = value
        //一旦resolve执行，调用成功数组的函数
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    //失败
    let reject = (error) => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        //存储成功的值
        this.error = error
        //一旦reject执行，调用失败数组的函数
        this.onRejectedCallbacks.forEach(fn => fn())
       
      }
    }
    //如果executor执行报错，执行rejected
    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
    
  }

  then(onFulfilled,onRejected) {
    // onFulfilled如果不是函数，就忽略onFulfilled，直接返回value
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    // onRejected如果不是函数，就忽略onRejected，直接扔出错误
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
    let promise2 = new MyPromise((resolve,reject)=> {
      //状态为fulfilled，执行该函数传入成功的值
      if (this.state === 'fulfilled') {
        // let x = onFulfilled(this.value)
        // resolvePromise(promise2, x, resolve, reject)
        // 异步

        setTimeout(() => {
          try {
            let x = onFulfilled(this.value)//每个then都会返回一个promise,因为只有promise菜可以继续执行then方法，也就是链式调用，x可能是一个值，也可能是一个函数或者返回一个promise,所以要多x做一系列的判断，并且返回的结果不能是promise2(也就是他自己)
            resolvePromise(promise2, x, resolve, reject)
          } catch (error) {
            reject(error)
          }
        },0)
      }
      //状态为rejected，执行该函数传入失败的原因
      if (this.state === 'rejected') {
        // 异步
        setTimeout(() => {
          try {
            let x = onRejected(this.error)
            resolvePromise(promise2, x, resolve, reject)
          } catch (error) {
            reject(error)
          }
        }, 0)
       
      }
      //当状态为pending时
      if (this.state === 'pending') {//当状态时等待状态的时候，要将成功和失败的回调存入对应的数组中。之所以存数组是因为成功或者失败的回调不止一个
        this.onResolvedCallbacks.push(() => {
          // 异步
          setTimeout(() => {
           try {
             let x = onFulfilled(this.value)
             resolvePromise(promise2, x, resolve, reject)
           } catch (error) {
             reject(error)
           }
          },0)
          
        })
        this.onRejectedCallbacks.push(() => {
          // 异步
          setTimeout(() => {
            try {
              let x = onRejected(this.error)
              resolvePromise(promise2, x, resolve, reject)
            } catch (error) {
              reject(error)
            }
          },0)
          
        })
      }
    })
    return promise2
  }
}
```

###### 扩展方法

`catch`,`reject`,`resolve`

```javascript
class MyPromise {
    constructor(executor){
        .....
    }
    then(){
        ....
    }
    catch(callback){
        return this.then(null,callback)
    }
    resolve(value){
        return new MyPromise((resolve,reject)=> {
            resolve(value)
        })
    }
    reject(err){
         return new MyPromise((resolve,reject)=> {
            reject(err)
        })
    }
    //all方法(获取所有的promise，都执行then，把结果放到数组，一起返回)
    all(promises) {
        //在判断一下是不是数组不是数组抛出异常，，，
        let arr =[]
        let i = 0
        function processData(index,data){
            arr[index] = data
            i++
            if(i === promises.length){
                resolve(arr)
            }
        }
        return new MyPromise((resolve,reject) => {
            for(let i = 0,len=promises.length;i< len;i++){
                promises[i].then(data => {
                    processData(i,data)
                },reject)
            }
        })
    }
}
```



