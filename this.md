##### **`this`**

call/apply/bind可以显示绑定, 这里就不说了

隐试绑定

- 全局上下文
- 直接调用函数
- 对象.方法的调用
- DOM事件绑定（特殊）
- new构造函数绑定
- 箭头函数

###### **全局上下文**

------

全局上下文默认指向`window`严格模式下指向`undefined`

###### 直接调用函数

------



```javascript
let obj = {
  a: function() {
    console.log(this);
  }
}
let func = obj.a;
func();//window
```

这种情况数直接调用。this相当于全局上下文

###### 对象.方法的相识调用

------



```javascript
let obj = {
  a: function() {
    console.log(this);
  }
}
obj.a;//this指向obj
```

这就是`对象.方法`的情况，`this`指向这个对象

###### DOM事件绑定

------

onclick和addEventerListener中 this 默认指向绑定事件的元素。

IE比较奇异，使用attachEvent，里面的this默认指向window。

###### `new`构造函数

此时构造函数中的this指向实例对象。

###### 箭头函数

------

箭头函数没有`this`,因此也不能绑定。里面的`this`会指向当前最近的非箭头函数的`this`,找不到就指是`window`（严格模式下就是window）

```javascript
let obj = {
  a: function() {
    let do = () => {
      console.log(this);
    }
    do();
  }
}
obj.a(); // 找到最近的非箭头函数a，a现在绑定着obj, 因此箭头函数中的this是obj
```

> `优先级：new>call,apply,bind>对象.方法>直接调用`