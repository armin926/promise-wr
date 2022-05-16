:cherry_blossom: 实现 then 方法
------------------------------
因为 `then` 是在创建实例后再进行调用的，因此我们再创建一个 **类方法**，可千万不要创建在 `constructor` 里面了~ :smile:   
`then`方法可以传入两个参数，这两个参数都是函数，一个是当状态为`fulfilled 成功` 时执行的代码，另一个是当状态为 `rejected 拒绝` 时执行的代码。    
虽然很多人可能一直只用一个函数参数，但不要忘记这里是两个函数参数:astonished:    
因此我们就可以先给手写的`then`里面添加**两个参数**：    
 * 一个是 `onFulfilled` 表示 `“当状态为成功时”`
 * 另一个是 `onRejected` 表示 `“当状态为拒绝时”`    
``` js
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
+   then(onFulfilled, onRejected) {}
}
```
### 1. 状态不可变
-----
让我们先看看`原生Promise`产生的结果：
``` js
let promise = new Promise((resolve, reject) => {
    resolve('这次一定')
    reject('下次一定')
})

promise.then(
    result => {
        console.log('fulfilled', result);
    },
    reason => {
        console.log('rejected', reason);
    }
)
```
![执行图](/images/img-4.png 'promise执行')    
可以看到控制台只显示了一个 `console.log` 的结果，**证明 `Promise`只会执行`成功状态` 或者 `拒绝状态` 的其中一个**
因此我们在手写的时候就必须进行判断：:eyes:

* 如果当前实例的 `PromiseState` 状态属性为 `fulfilled 成功` 的话，我们就执行传进来的 `onFulfilled` 函数，并且为`onFulfilled`函数传入前面保留的`PromiseResult`属性值：
``` js
    then(onFulfilled, onRejected) {
+       if (this.PromiseState === myProise.FULFILLED) {
+           onFulfilled(this.PromiseResult);
+       }
    }
```
* 如果当前实例的 `PromiseState` 状态属性为 `rejected` 拒绝 的话，我们就执行传进来的 `onRejected` 函数，并且为`onRejected`函数传入前面保留的`PromiseResult`属性值：
``` js
    then(onFulfilled, onRejected) {
        if (this.PromiseState === myProise.FULFILLED) {
            onFulfilled(this.PromiseResult);
        }
+       if (this.PromiseState === myPromise.REJECTED) {
+           onRejected(this.PromiseResult);
+       }
    }
```
定义好了判断条件以后，我们就来测试一下代码，也是一样，在实例 :chestnut: 上使用`then`方法：
``` js
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
    then(onFulfilled, onRejected) {
        if (this.PromiseState === myPromise.FULFILLED) {
            onFulfilled(this.PromiseResult);
        }
        if (this.PromiseState === myPromise.REJECTED) {
            onRejected(this.PromiseResult);
        }
    }
}


// 测试代码
let promise1 = new myPromise((resolve, reject) => {
  resolve('这次一定');
   reject('下次一定');
})
   promise1.then(
       result => {
           console.log(result)
       },
       reason => {
           console.log(reason.message)
       }
   )
```
可以看到控制台只显示了一个`console.log`的结果：`这次一定` :grinning:，证明我们已经实现了 `promise的状态不可变` :clap::clap::clap:

写到这里并没有报错，也就是我们`暂时安全`了，为什么说`暂时安全`呢？

因为这里还有很多没有完善的地方，手写Promise的时候，有一个难点就在于有很多地方需要和原生一样严谨，也就是说原生的Promise会考虑很多特殊情况~

我们在实际运用时可能暂时不会碰到这些情况，可是当我们遇到的时候 **却不知底层的原理**，`这就是为什么我们要知道如何手写Promise`

加油！！:muscle:

### 2.执行异常 throw
----------------------
在`new Promise`的时候，执行函数里面如果抛出错误，是会触发`then`方法的第二个参数，即`rejected`状态的回调函数

也就是在原生的Promise里面，`then`方法的第二个参数，即`rejected`状态的回调函数可以把错误的信息作为内容输出出来

到这里，有的同学可能会说，执行异常抛错，不是用catch()方法去接吗？为什么这里又说 `是会触发then方法的第二个参数，即rejected状态的回调函数？`:dizzy_face:
那我们就说道说道吧:yum:：

`catch()` 方法返回一个`Promise`，并且处理拒绝的情况。它的行为与调用`Promise.prototype.then(undefined, onRejected)` 相同。

事实上, calling `obj.catch(onRejected)` 内部calls `obj.then(undefined, onRejected)`。(这句话的意思是，我们显式使用`obj.catch(onRejected)`，内部实际调用的是`obj.then(undefined, onRejected))`

`Promise.prototype.catch()`方法是`.then(null, rejection)`或`.then(undefined, rejection)`的别名，用于指定发生错误时的回调函数。

``` js
p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同于
p.then(
    null,
    err=> {console.log(err)}
) 

// 等同于
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
```

**一般来说，不要在`then()`方法里面定义 Reject 状态的回调函数（即`then`的第二个参数），总是使用`catch`方法。**

``` js
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });
  
// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

**回到正题**

原生Promise在`new Promise`的时候，执行函数里面如果抛出错误，是会触发`then`方法的第二个参数 `(即rejected状态的回调函数)`，把错误的信息作为内容输出出来:
``` js
let promise = new Promise((resolve, reject) => {
    throw new Error('白嫖不成功');
})

promise.then(
    result => {
        console.log('fulfiiled:', result)
    },
    reason => {
        console.log('rejected:', reason)
    }
)
```
![执行图](/images/img-5.png 'promise执行')

但是如果我们在手写这边写上同样道理的测试代码，很多人就会忽略这个细节:disappointed_relieved:：
``` js
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
    then(onFulfilled, onRejected) {
        if (this.PromiseState === myPromise.FULFILLED) {
            onFulfilled(this.PromiseResult);
        }
        if (this.PromiseState === myPromise.REJECTED) {
            onRejected(this.PromiseResult);
        }
    }
}


// 测试代码
let promise1 = new myPromise((resolve, reject) => {
   throw new Error('白嫖不成功');
})
promise1.then(
   result => {
       console.log('fulfiiled:', result)
   },
   reason => {
       console.log('rejected:', reason)
   }
)
```
![执行图](/images/img-6.png 'promise执行')

可以发现报错了:cold_sweat:，没有捕获到错误，没有把内容输出出来

我们可以在执行`resolve()`和`reject()`之前用`try/catch`进行判断，在`构造函数 constructor`里面完善代码，判断生成实例的时候是否有报错 :mag:：

  * 如果没有报错的话，就按照正常执行`resolve()`和`reject()`方法
  * 如果报错的话，就把错误信息传入给`reject()`方法，并且直接执行`reject()`方法

``` js
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
+       try {
            func(this.resolve.bind(this), this.reject.bind(this));
+       } catch (error) {
+           this.reject(error)
+       }
    }
```

◾ 注意这里不需要给`reject()`方法进行`this`的绑定了，因为这里是直接执行，而不是创建实例后再执行。

▪ `func(this.resolve.bind(this), this.reject.bind(this));` 这里的`this.reject`意思是：把类方法`reject()`作为参数 传到构造函数`constructor` 里要执行的`func()`方法里，只是一个参数，并不执行，只有创建实例后调用`reject()`方法的时候才执行，此时`this`的指向已经变了，所以想要正确调用`myPromise`的`reject()`方法就要通过`.bind(this))`改变`this`指向。

▪ `this.reject(error)`，这里的`this.reject()`，是直接在构造函数里执行类方法，`this`指向不变，`this.reject()`就是直接调用类方法`reject()`，所以不用再进行`this`绑定。

◾ 这里考察了this绑定的一个细节:mag:：
`call、apply和bind`都可以改变函数体内部 this 的指向，**但是 `bind` 和 `call/apply` 有一个很重要的区别：一个函数被 `call/apply` 的时候，会立即执行函数，但是 `bind` 会创建一个新函数，不会立即执行。**

这就是前面为什么说， this.reject.bind(this)只是作为参数，并没有直接执行的原因了~:smile:

**回到正文**

结合前面的讲解，刷新一下控制台，我们可以看到手写这边已经没有报错了:clap::clap::clap::
![执行图](/images/img-7.png 'promise执行')

### 3.参数校验
---------------------------

大家觉得目前代码是不是没问题了？可以进行下一步了？

如果你觉得是的话就又掉坑了~:see_no_evil:

原生Promise里**规定`then`方法里面的两个参数如果不是函数的话就要被忽略**，我们就故意在原生代码这里不传入函数作为参数：

``` js
let promise = new Promise((resolve, reject) => {
    throw new Error('白嫖不成功');
})

promise.then(
    undefined,
    reason => {
        console.log('rejected:', reason)
    }
)
```

运行以后我们发现在这里执行是没有问题的，我们再以同样类似的不传 **函数参数** 的代码应用在 **手写代码** 上面，大家想想会不会有什么问题？来看看结果会怎样？🧐

![执行图](/images/img-8.png 'promise执行')

结果就是 `Uncaught TypeError: onFulfilled is not a function`。浏览器帮你报错了，这不是我们想要的~:disappointed_relieved:

**`Promise` 规范如果 `onFulfilled` 和 `onRejected` 不是函数，就忽略他们，所谓“忽略”并不是什么都不干，对于`onFulfilled`来说“忽略”就是将`value`原封不动的返回，对于`onRejected`来说就是返回`reason`，`onRejected`因为是错误分支，我们返回`reason`应该`throw`一个`Error`:**

这里我们就可以用 `条件运算符`，我们在进行`if`判断之前进行预先判断：

▪ 如果`onFulfilled`参数是一个函数，就把原来的`onFulfilled`内容重新赋值给它，如果`onFulfilled`参数不是一个函数，就将`value`原封不动的返回

▪ 如果`onRejected`参数是一个函数，就把原来的`onRejected`内容重新赋值给它，如果`onRejected`参数不是一个函数，就`throw`一个`Error`
``` js
class myPromise {
	...
    then(onFulfilled, onRejected) {
+       onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
+       onRejected = typeof onRejected === 'function' ? onRejected : reason => {
            throw reason;
        };
        if (this.PromiseState === myPromise.FULFILLED) {
            onFulfilled(this.PromiseResult);
        }
        if (this.PromiseState === myPromise.REJECTED) {
            onRejected(this.PromiseResult);
        }
    }
}
```