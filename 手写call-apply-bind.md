#### 手写`call`,`apply`,`bind`实现及原理

##### 原理

###### `call`

***

> call 接收多个参数，第一个为函数上下文也就是this，后边参数为函数本身的参数。

```javascript
const obj = {
  name:'叶俊宽'
}

function getName(firstName,lastName) {
  console.log(this)
  console.log(firstName,lastName,this.name)
}

getName('叶','俊宽')//此时的this在node环境中是只想global的,this.name 是undefined

getName.call(obj,'叶','俊宽')//此时的this指向obj,此时的this.name 是 叶俊宽
```

###### `apply`

***

> apply接收两个参数，第一个参数为函数上下文this，第二个参数为函数参数只不过是通过一个数组的形式传入的。

```javascript
const obj = {
  name:'叶俊宽'
}

function getName(firstName,lastName) {
  console.log(this)
  console.log(firstName,lastName,this.name)
}

getName('叶','俊宽')//此时的this在node环境中是只想global的,this.name 是undefined

getName.apply(obj,['叶','俊宽'])//此时的this指向obj,此时的this.name 是 叶俊宽
```

###### `bind`

***

>bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )

`bnd`特点

1. 返回一个函数
2. 可以传入一个参数

```javascript
const obj = {
  name: '叶俊宽'
}

function getName(firstName,lastName,target){
  console.log(this)
  console.log(firstName,lastName,this.name,target)
}

getName('叶','俊宽','财富自由')

getName.bind(obj)//不会立即执行

const fn = getName.bind(obj,'叶','俊宽','财富自由')
fn()//叶 俊宽 叶俊宽 财富自由

const fn1 = getName.bind(obj)
fn1('叶','俊宽','财富自由')//叶 俊宽 叶俊宽 财富自由
// 也可以这样用，参数可以分开传。bind后的函数参数默认排列在原函数参数后边
const fn2 = getName.bind(obj,'叶')
fn2('俊宽','叶俊宽')//叶 俊宽 叶俊宽 财富自由
```

##### 手动实现

###### `call`

```javascript
const Person = {
  name:'叶俊宽',
  say: function(){
    console.log(`我叫${this.name}`)
  }
}

const Person1 = {
  name: '叶飞飞'
}

Person.say.call(Person1)//我叫叶飞飞
```

在这里注意两点

1. `call`改变了`this`指向，指向了`Person1`
2. `say`函数执行了

- 模拟实现第一步

  ***

  当调用`call`的时候，把`Person1`对象改造成了下面的样子

  ```javascript
  const Person1 = {
    name:'叶飞飞',
    say: function(){
      console.log(`我叫${this.name}`)
    }
  }
  Person1.say()
  ```

  这个时候 this 就指向了 foo

  但是这样却给 foo 对象本身添加了一个属性

  不过也不用担心，我们用 delete 再删除

  模拟步骤如下

  - 将函数设置为对象的属性
  - 执行该函数
  - 删除该函数

  ```javascript
  Person1.fn = Peson.fn
  Person1.fn()
  delete Person.fn
  ```

  根据这个写出第一个`call`函数

  ```javascript
  Function.prototype.myCall = function(context){
    console.log(this)
    context.say = this
    context.say()
    delete context.say
  }
  Person.say.myccall(Person1) //我叫叶飞飞
  ```

- 模拟实现第二步

  ***

  添加参数

  ```javascript
  const Person = {
    name:'叶俊宽',
    say: function(age,height){
      console.log(`我叫${this.name}身高:${height}年龄：${age}`,)
    }
  }
  
  const Person1 = {
    name: '叶飞飞'
  }
  
  Person.say.call(Person1,18,180)//我叫叶飞飞身高:180年龄：18
  ```

  注意：传入的参数并不确定

  从`arguments`中获取，取出第二个参数到最后一个参数存入一个数组中

  ```javascript
  function getParams() {
    // 以上个例子为例，此时的arguments为：
    // arguments = {
    //      0: foo,
    //      1: 'kevin',
    //      2: 18,
    //      length: 3
    // }
    // 因为arguments是类数组对象，所以可以用for循环
    var args = [];
    for (var i = 1, len = arguments.length; i < len; i++) {
      args.push("arguments[" + i + "]");
    }
  
    // 执行后 args为 ["arguments[1]", "arguments[2]", "arguments[3]"]
  }
  ```

  ps:直接用slice方法会快点

  ```javascript
  const args = arguments.slice(1)
  ```

  还有一个问题要解决就是,这个参数数组放到要执行的函数的参数里面去。

  ```javascript
  function mySymbol(obj) {
    // 不要问我为什么这么写，我也不知道就感觉这样nb
    let unique = (Math.random() + new Date().getTime()).toString(32).slice(0, 8);
    // 牛逼也要严谨
    if (obj.hasOwnProperty(unique)) {
      return mySymbol(obj); //递归调用
    }
    return unique;
  }
  ```

  第二版如下

  ```java
  Function.prototype.myCall = function (context) {
    console.log(this);
    context.say = this;
    const fn = mySymbol(context)
    const args = [...arguments].slice(1);
    context[fn](...args)
    delete context.say;
  };
  
  ```

- 模拟实现第三步

  ***

  `this`参数可以传入`null`,`undefined`

  这个时候还得做一层处理

  第三版本代码如下

  ```javascript
  Function.prototype.myCall = function (context) {
    console.log(this);
    context.say = this || window;
    const fn = mySymbol(context)
    const args = [...arguments].slice(1);//[...arguments]将类数组转成真正的数组
    context[fn](...args)
    delete context.say;
  };
  ```

###### `apply`

和`call`类似，不过第二个参数是一个数组，直接贴代码

```javascript
Function.prototype.myCall = function (context) {
  console.log(this);
  context.say = this || window;
  const fn = mySymbol(context)
  const args = [...arguments].slice(1);
  context[fn](args)
  delete context.say;
};
```

###### `bind`

这个和call、apply区别还是很大的，容我去抽根烟回来收拾它 还是老套路先分析bind都能干些什么，有什么特点 

1. 函数调用，改变this
2. 返回一个绑定this的函数 
3. 接收多个参数 、支持柯里化形式传参 fn(1)(2)

返回函数模拟实现

***

```javascript
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

// 返回了一个函数
var bindFoo = bar.bind(foo); 

bindFoo(); // 1
```

所以第一版代码

```javascript
Function.prototype.myBind = function(contenxt){
  return () => {
    return this.apply(contenxt)
  }
}
```

此外，之所以 `return self.apply(context)`，是考虑到绑定函数可能是有返回值的，依然是这个例子：

```javascript
var foo = {
    value: 1
};

function bar() {
	return this.value;
}

var bindFoo = bar.bind(foo);

console.log(bindFoo()); // 1
```

传参的模拟实现

***

```javascript
const foo = {
  value: 1
};

function bar(name,age) {
  console.log(this.value);
  console.log(name)
  console.log(age)
}

// 返回了一个函数
const bindFoo = bar.bind(foo,'叶俊宽','18'); 

bindFoo(); // 1 叶俊宽 18
```

函数需要传 name 和 age 两个参数，竟然还可以在 bind 的时候，只传一个 name，在执行返回的函数的时候，再传另一个参数 age!

第二版

```javascript
Function.prototype.myBind = function(contenxt){
  const self = this
  //获取bind2函数从第二个参数到最后一个参数
  const args = [...arguments].slice(1)

  return function()  {//不能用箭头函数，箭头函数没有arguments
    const bindArgs = [...arguments].slice()//这个时候的arguments是指bind返回的函数传入的参数
    return self.apply(contenxt,[...args,...bindArgs])
  }
}
```

构造函数效果模拟实现

完成了这两点，最难的部分到啦！因为 bind 还有一个特点，就是

> 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

也就是说当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。举个例子：

```javascript
const value = 1
const fool = {
  value: 2
}

const bar = function(name,age){
  this.habit = '写代码'
  console.log(this.value)
  console.log(name)
  console.log(age)
}

bar.prototype.friend = '谢致远'

const bindFool = bar.bind(fool,'叶俊宽')
const obj = new bindFool('18') //undefind 叶俊宽 18
console.log(obj.habit)//写代码
console.log(obj.friend)//谢致远
```

注意：尽管在全局和 foo 中都声明了 value 值，最后依然返回了 undefind，说明绑定的 this 失效了，如果大家了解 new 的模拟实现，就会知道这个时候的 this 已经指向了 obj。

第三版

```javascript
Function.prototype.myBind = function(context) {
  const self = this
  const args = [...arguments].slice(1)
  const fBound = function() {
    const bingArgs = [...arguments].slice()
    // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
    // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
    // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return self.apply(this instanceof fBound ? this : context,[...args,...bingArgs])
  }

  fBound.prototype = this.prototype
  return fBound
}
```

构造函数优雅实现

***

```javascript
Function.prototype.myBind = function(context){
  const self = this
  const args = [...arguments].slice(1)
  const cFn = function(){}
  const fBound = function() {
    const bindArgs = [...arguments].slice()
    return self.apply(this instanceof cFn ? this : context,[...args,...bindArgs])
  }
  cFn.prototype = this.prototype
  fBound.prototype = cFn.prototype
  return fBound
}
```

最后做一下异常处理，如果传入的不是函数的话

```javascript
if (typeof this !== "function") {
  throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
}
```

最终的代码是

```javascript
Function.prototype.myBind = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

