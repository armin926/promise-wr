<!--
 * @Descripttion: 实现 resolve 和 reject
 * @Author: armin
 * @Date: 2022-02-15 14:32:36
 * @LastEditors: armin
 * @LastEditTime: 2022-02-15 17:19:39
-->
:fire:实现 resolve 和 reject
------------------------
原生的promise里面可以传入 `resolve` 和 `reject` 两个参数
```js
    let promise = new Promise((resolve, reject) => {})
```
所以我们在手写的Promise也要允许传入这两个参数：
```js
    class myPromise {
      constructor(func) {
    +    func(resolve, reject)
      }
    }
```
可是这样写的话有一个很明显的问题:frowning:，由于`resolve`和`reject`还没有定义，所以不知道哪里调用这两个参数   
因此就需要创造出这两个对象:smiley:   
`resolve()`和`reject()`是以函数形式执行的，所以我们可以用类方法的形式来创建这两个函数
```js
    class myPromise {
      constructor(func) {
    +    func(this.resovle, this.reject) // 注意这里和上面的区别，需要用 this 来调用 class 自身的方法
      }
    +  resovle() {}
    +  reject() {}
    }
```
这里的`resolve()`和`reject()`方法应该如何执行呢？里面应该写什么内容呢？:hushed:   
这里就需要用到状态了:stuck_out_tongue_winking_eye:  
### :ocean:管理状态和结果
***
promise有三种状态：分别是 **pending**、**fulfilled** 和 **rejected**   
- 初始的时候是**pending**
- **pending**可以转为**fulfilled**状态，但是不能逆转
- **pending**也可以转为**rejected**状态，但是也不能逆转
- 这里**fulfilled**和**rejected**也不能互转   
![状态图](/images/img-1.png 'promise状态图')   
因此我们需要提前把状态定义好   
```js
    class myPromise {
    +  static PENDING = 'pending'
    +  static FULFILLED = 'fulfilled'
    +  static REJECTED = 'rejected'
      constructor(func) {
        func(this.resolve, this.reject)
      }
      resolve() {}
      reject() {}
    }
```
创建了状态属性后还需要为每个实例添加一个`状态属性`，并且默认为`pending`状态，**这样在每一个实例被创建以后就会有自身的状态属性可以进行判断和变动了**
```js
    class myPromise {
      static PENDING = 'pending'
      static FULFILLED = 'fulfilled'
      static REJECTED = 'rejected'
      constructor(func) {
    +   this.PromiseState = myPromise.PENDING
        func(this.resolve, this.reject)
      }
      resolve() {}
      reject() {}
    }
```
那么在执行 `resolve()` 的时候就需要判断状态是否为 `待定pending`，如果是 `待定pending` 的话就把状态改为 `成功fulfilled`。同理，为`reject()` 添加 :muscle:
```js
    class myPromise {
      static PENDING = 'pending'
      static FULFILLED = 'fulfilled'
      static REJECTED = 'rejected'
      constructor(func) {
        this.PromiseState = myPromise.PENDING
        func(this.resolve, this.reject)
      }
      resolve() {
    +   if(this.PromiseState === myPromise.PENDING) {
    +     this.PromiseState = myPromise.FULFILLED
    +   }
      }
      reject() {
    +   if(this.PromiseState === myPromise.PENDING) {
    +     this.PromiseState = myPromise.REJECTED
    +   }
    }
```
#### 执行 `resovle()` 和 `reject()` 可以传参
先让我们回忆一下原生Promise，在执行 `resolve()` 和 `reject()` 的时候都是可以传入一个参数，并且可以在后面使用这个参数。
```js
    let promise = new Promise((resolve,reject) => {
      resolve('我来试试')
    })
```
我们把结果参数命名为 `PromiseResult` (和原生Promise保持一致)，让每个实例都有 `PromiseResult` 属性，并且给他们都赋值 `null`。接下来，我们就可以给`resolve()` 和 `reject()` 添加参数，并且把参数赋值给实例的 `PromiseResult` 属性：
```js
    class myPromise {
      static PENDING = 'pending'
      static FULFILLED = 'fulfilled'
      static REJECTED = 'rejected'
      constructor(func) {
        this.PromiseState = myPromise.PENDING
    +   this.PromiseResult = null
        func(this.resolve, this.reject)
      }
    + resolve(result) {
        if(this.PromiseState === myPromise.PENDING) {
          this.PromiseState = myPromise.FULFILLED
    +     this.PromiseResult = result
        }
      }
    + reject(reason) {
        if(this.PromiseState === myPromise.PENDING) {
          this.PromiseState = myPromise.REJECTED
    +     this.PromiseResult = reason
        }
    }
```
### :point_right: this 指向问题
***
现在的代码看起来风平浪静，但这里很容易犯错 :disappointed_relieved:   
我们先来 new 一个实例 :chestnut: 执行一下代码
```js
     // 测试代码
  +  let promise1 = new myPromise((resolve, reject) => {
  +    resolve('我来试试')
  +  })
```
运行上面代码报错:tada:   
`Uncaught TypeError: Cannot read property 'PromiseState ' of undefined`   
奇了怪了:anguished:，`PromiseState` 属性我们已经创建了，怎么会是 `undefined`:unamused:   
:mag:这时候，**请！注！意！！！**
```js
    resolve(result) {
      // 前面的 this 关键字
➡    if (this.PromiseState === myPromise.PENDING) {
➡      this.PromiseState = myPromise.FULFILLED;
        this.PromiseResult = result;
      }
    }
    reject(reason) {
      // 前面的 this 关键字
➡    if (this.PromiseState === myPromise.PENDING) {
➡      this.PromiseState = myPromise.REJECTED;
        this.PromiseResult = reason;
      }
    }
```
所以现在只有一种可能，调用`this.PromiseState`的时候并没有调用`constructor`里面的`this.PromiseState`，这里的`this`跟丢咯~:joy::joy::joy:   
我们在`new`一个新实例的时候执行的是`constructor`里的内容，也就是说`constructor`里的`this`确实是新实例的，但现在我们是在新实例被创建后再在外部环境下执行`resolve()`方法的，这里的`resolve()`看着像是和实例一起执行的，其实不然，也就**相当于不在`class`内部使用这个`this`，而我们没有在外部定义任何 `PromiseState` 变量，因此这里会报错**   
解决`class`的`this`指向问题一般会用箭头函数，`bind`或者`proxy`，这里我们就使用 `bind` 来绑定 `this` :smiley_cat: ： 
```js
  class myPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
+       func(this.resolve.bind(this), this.reject.bind(this));
    }
    resolve(result) {
        if (this.PromiseState === myPromise.PENDING) {
            this.PromiseState = myPromise.FULFILLED;
            this.PromiseResult  = result;
        }
    }
    reject(reason) {
        if (this.PromiseState === myPromise.PENDING) {
            this.PromiseState = myPromise.REJECTED;
            this.PromiseResult = reason;
        }
    }
  }

// 测试代码
let promise1 = new myPromise((resolve, reject) => {
    resolve('我来试试');
})
``` 
（:mega:PS：如果这里有点蒙圈的话，可以参考一下网上有关 `this` 指向的文章，这里就不过多解释了）   
对于`resolve`来说，这里就是给实例的`resolve()`方法绑定这个`this`为当前的实例对象，并且执行`this.resolve()`方法，`reject`同理：

![示意图](/images/img-2.png 'this指向示意图')   

现在再来测试一下我们的代码：   
```js
class myPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
        func(this.resolve.bind(this), this.reject.bind(this));
    }
    resolve(result) {
        if (this.PromiseState === myPromise.PENDING) {
            this.PromiseState = myPromise.FULFILLED;
            this.PromiseResult = result;
        }
    }
    reject(reason) {
        if (this.PromiseState === myPromise.PENDING) {
            this.PromiseState = myPromise.REJECTED;
            this.PromiseResult = reason;
        }
    }
}

// 测试代码
let promise1 = new myPromise((resolve, reject) => {
    resolve('我来试试');
})
console.log(promise1); 
// myPromise {PromiseState: 'fulfilled', PromiseResult: '我来试试'}
let promise2 = new myPromise((resolve, reject) => {
    reject('我再来试试');
})
console.log(promise2); 
// myPromise {PromiseState: 'rejected', PromiseResult: '我再来试试'}
```
再来看看原生Promise的执行情况：
```js
let promise1 = new Promise((resolve,reject) => {
  resolve('我来试试')
})
console.log(promise1)
let promise2 = new Promise((resolve,reject) => {
  reject('我再来试试')
})
console.log(promise2)
```
![原生Promise](/images/img-3.png '原生Promise')  
说明执行结果完全符合我们的预期，是不是觉得离成功又进了一步呢~ :clap::clap::clap:   
下一步就需要实现`then`方法了...