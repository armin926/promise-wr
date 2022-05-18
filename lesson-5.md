:sweat_drops: 实现异步
----------
### 1.添加定时器
----------------
可以说我们在手写的代码里面依旧没有植入异步功能，毕竟最基本的`setTimeout`我们都没有使用，但是我们必须先了解一下原生Promise的一些`运行顺序规则`。

在这里我们为原生代码添加上步骤信息：
``` js
console.log(1);

let promise = new Promise((resolve, reject) => {
    console.log(2);
    resolve('这次一定');
})

promise.then(
    result => {
        console.log('fulfilled:', result);
    },
    reason => {
        console.log('rejected:', reason)
    }
)

console.log(3);
```
输出顺序为：
``` js
1
2
3
fulfilled: 这次一定
```

我们用同样的测试代码应用在**手写代码**上面，我们会发现有些不同了:open_mouth:，输出顺序为：
``` js
1
2
fulfilled: 这次一定
3
```
其实问题很简单，就是我们上面说的 **没有设置异步执行**:frowning:

我们二话不说直接给`then`方法里面添加`setTimeout`就可以了:sunglasses:，**需要在进行`if`判断以后再添加`setTimeout`，要不然状态不符合添加异步也是没有意义的**，然后在`setTimeout`里执行传入的函数参数：

``` js
class myPromise {
	...
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => {
            throw reason;
        };
        if (this.PromiseState === myPromise.FULFILLED) {
+           setTimeout(() => {
                onFulfilled(this.PromiseResult);
+           });
        }
        if (this.PromiseState === myPromise.REJECTED) {
+           setTimeout(() => {
                onRejected(this.PromiseResult);
+           });
        }
    }
}
```
现在我们重新测试一下代码，得到的输出顺序为：
``` js
1
2
3
fulfilled: 这次一定
```
**在这里我们解决异步的方法是给`resolve`和`reject`添加`setTimeout`，但是为什么要这么做呢？**

这里就要讲到[Promise/A+规范](https://promisesaplus.com/)了

规范 `2.2.4`：

    onFulfilled or onRejected must not be called until the execution context stack contains only platform code. [3.1].

译文：

2.2.4 `onFulfilled` 和 `onRejected` 只有在`执行环境`堆栈仅包含平台代码时才可被调用 `注1`

规范对2.2.4做了注释：

    3.1 Here “platform code” means engine, environment, and promise implementation code. In practice, this 
    requirement ensures that onFulfilled and onRejected execute asynchronously, after the event loop turn in 
    which then is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism 
    such as setTimeout or setImmediate, or with a “micro-task” mechanism such as MutationObserver or 
    process.nextTick. Since the promise implementation is considered platform code, it may itself contain a task-
    scheduling queue or “trampoline” in which the handlers are called.

译文：
**3.1 这里的平台代码指的是引擎、环境以及 promise 的实施代码。实践中要确保 `onFulfilled` 和 `onRejected` 方法异步执行，且应该在 `then` 方法被调用的那一轮事件循环之后的新执行栈中执行。这个事件队列可以采用“宏任务（macro-task）”机制，比如 `setTimeout` 或者 `setImmediate`； 也可以采用“微任务（micro-task）”机制来实现， 比如 `MutationObserver` 或者 `process.nextTick`。** 由于 promise 的实施代码本身就是平台代码（译者注： 即都是 JavaScript），故代码自身在处理程序时可能已经包含一个任务调度队列或『跳板』)。

这里我们用的就是规范里讲到的 “宏任务” `setTimeout`。

### 2.回调保存
---------------
异步的问题真的解决了吗？现在又要进入Promise另一个难点了，大家务必竖起耳朵啦:stuck_out_tongue_winking_eye:

我们来给原生的Promise里添加`setTimeout`，使得`resolve`也异步执行，那么就会出现一个问题了，`resolve`是异步的，`then`也是异步的，究竟谁会先被调用呢？

``` js
console.log(1);
let promise = new Promise((resolve, reject) => {
    console.log(2);
+   setTimeout(() => {
        resolve('这次一定');
+       console.log(4);
+   });
})
promise.then(
    result => {
        console.log('fulfilled:', result);
    },
    reason => {
        console.log('rejected:', reason)
    }
)
console.log(3);
```
输出顺序为：
``` js
1
2
3
4
fulfilled: 这次一定
```

这里涉及到了浏览器的事件循环，`promise.then()` 和 `setTimeout()` 都是异步任务，但实际上异步任务之间并不相同，因此他们的执行优先级也有区别。不同的异步任务被分为两类：`微任务 (micro task)` 和 `宏任务 (macro task)`。

  - `setTimeout()` 属于宏任务
  - `promise.then()` 属于微任务

我们只需记住 **当 当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行。**

我们将同样的代码应用到手写的部分发现，`fulfilled: 这次一定` 并没有输出

我们可以先猜测一下，没有输出的原因很可能是因为`then`方法没有被执行，也就是说很可能没有符合的条件，再换句话说可能没有符合的状态

那么我们就在三个位置分别输出当前的状态，这样分别来判断哪个位置出了问题:

``` js
class myPromise {
	...
}

// 测试代码
console.log(1);
let promise1 = new myPromise((resolve, reject) => {
    console.log(2);
    setTimeout(() => {
+       console.log('A',promise1.PromiseState);
        resolve('这次一定');
+       console.log('B',promise1.PromiseState);
        console.log(4);
    });
})
promise1.then(
    result => {
+       console.log('C',promise1.PromiseState);
        console.log('fulfilled:', result);
    },
    reason => {
        console.log('rejected:', reason)
    }
)
console.log(3);
```
输出结果为：
``` js
1
2
3
A pending
B fulfilled
4
```

发现只有两组状态被输出，这两组都在`console.log(4)`前被输出，证明`setTimeout`里面的状态都被输出了，只有`then`里面的状态没有被输出

这基本就可以确定是因为`then`里的状态判断出了问题

这里涉及到事件循环，我们详细解读一下：

▪ 首先，执行`console.log(1)`，输出`1`

▪ 第二步，创建promise，执行函数体里的`console.log(2)`，输出`2`

▪ 第三步，遇到`setTimeout`，`setTimeout`是宏任务，将`setTimeout`加入宏任务队列，等待执行

▪ 第四步，遇到`promise.then()`，`promise.then()`是微任务，将`promise.then()`加入微任务队列，等待执行

▪ 第五步，执行`console.log(3)`，输出`3`，此时当前执行栈已经清空

▪ 第六步，当前执行栈已经清空，先执行微任务队列的任务 `promise.then()`，发现promise的状态并没有改变，还是`pending`，所以没有输出。状态并没有改变的原因是：`resolve('这次一定')`是在`setTimeout`里的，但此时还没开始执行`setTimeout`，因为`setTimeout`是宏任务，宏任务在微任务后面执行

▪ 第七步，微任务队列已经清空，开始执行宏任务 `setTimeout`：
``` js
setTimeout(() => {
     console.log('A',promise1.PromiseState);
     resolve('这次一定');
     console.log('B',promise1.PromiseState);
     console.log(4);
});
```
▪ 第八步，执行 `console.log('A',promise1.PromiseState)`，此时promise状态还没发生变化，还是`pending`，所以输出 `A pending`

▪ 第九步，执行 `resolve('这次一定')`;，改变promise的状态为`fulfilled`

▪ 第十步，执行 `console.log('B',promise1.PromiseState)`，输出 `B fulfilled`

▪ 第十一步，执行 `console.log(4)`，输出`4`

    这里暂且认为我们写的promise.then()和原生一样，方便理解

分析完上面的代码，我们知道了，因为先执行了`then`方法，但发现这个时候状态依旧是 `pending`，而我们手写部分没有定义`pending`待定状态的时候应该做什么，因此就少了`fulfilled: 这次一定` 这句话的输出

所以我们就 **直接给`then`方法里面添加待定状态的情况就可以了**，也就是用`if`进行判断:
``` js
class myPromise {
	...
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => {
            throw reason;
        };
+       if (this.PromiseState === myPromise.PENDING) {
+ 		
+ 		}
        if (this.PromiseState === myPromise.FULFILLED) {
            setTimeout(() => {
                onFulfilled(this.PromiseResult);
            });
        }
        if (this.PromiseState === myPromise.REJECTED) {
            setTimeout(() => {
                onRejected(this.PromiseResult);
            });
        }
    }
}
```
但是问题来了，当`then`里面判断到 `pending` 待定状态时我们要干什么？

因为这个时候`resolve`或者`reject`还没获取到任何值，因此我们必须让`then`里的函数稍后再执行，等`resolve`执行了以后，再执行`then`

为了保留`then`里的函数，我们可以创建 `数组` 来 **保存函数**。

**为什么用 `数组` 来保存这些回调呢？因为一个promise实例可能会多次 `then`，也就是经典的 `链式调用`**，而且数组是先入先出的顺序

在实例化对象的时候就让每个实例都有这两个数组：

  - `onFulfilledCallbacks` ：用来 **保存成功回调**
  - `onRejectedCallbacks` ：用来 **保存失败回调**

``` js
class myPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
+       this.onFulfilledCallbacks = []; // 保存成功回调
+       this.onRejectedCallbacks = []; // 保存失败回调
        try {
            func(this.resolve.bind(this), this.reject.bind(this));
        } catch (error) {
            this.reject(error)
        }
    }
}
```

接着就完善`then`里面的代码，也就是当判断到状态为 `pending` 待定时，暂时保存两个回调，也就是说暂且把`then`里的两个函数参数分别放在两个数组里面：
``` js
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : () => {};
        onRejected = typeof onRejected === 'function' ? onRejected : () => {};
        if (this.PromiseState === myPromise.PENDING) {
+           this.onFulfilledCallbacks.push(onFulfilled);
+           this.onRejectedCallbacks.push(onRejected);
        }
        if (this.PromiseState === myPromise.FULFILLED) {
            setTimeout(() => {
                onFulfilled(this.PromiseResult);
            });
        }
        if (this.PromiseState === myPromise.REJECTED) {
            setTimeout(() => {
                onRejected(this.PromiseResult);
            });
        }
    }
```
数组里面放完函数以后，就可以完善`resolve`和`reject`的代码了

**在执行`resolve`或者`reject`的时候，遍历自身的`callbacks`数组**，看看数组里面有没有`then`那边 **保留** 过来的 **待执行函数**，**然后逐个执行数组里面的函数**，执行的时候会传入相应的参数：
``` js
class myPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
        this.onFulfilledCallbacks = []; // 保存成功回调
        this.onRejectedCallbacks = []; // 保存失败回调
        try {
            func(this.resolve.bind(this), this.reject.bind(this));
        } catch (error) {
            this.reject(error)
        }
    }
    resolve(result) {
        if (this.PromiseState === myPromise.PENDING) {
            this.PromiseState = myPromise.FULFILLED;
            this.PromiseResult = result;
+           this.onFulfilledCallbacks.forEach(callback => {
+               callback(result)
+           })
        }
    }
    reject(reason) {
        if (this.PromiseState === myPromise.PENDING) {
            this.PromiseState = myPromise.REJECTED;
            this.PromiseResult = reason;
 +          this.onRejectedCallbacks.forEach(callback => {
 +              callback(reason)
 +          })
        }
    }
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => {
            throw reason;
        };
        if (this.PromiseState === myPromise.PENDING) {
            this.onFulfilledCallbacks.push(onFulfilled);
            this.onRejectedCallbacks.push(onRejected);
        }
        if (this.PromiseState === myPromise.FULFILLED) {
            setTimeout(() => {
                onFulfilled(this.PromiseResult);
            });
        }
        if (this.PromiseState === myPromise.REJECTED) {
            setTimeout(() => {
                onRejected(this.PromiseResult);
            });
        }
    }
}
```
完善好代码后，让我们再来测试以下刚才的实例：
``` js
class myPromise {
	...
}


// 测试代码
console.log(1);
let promise1 = new myPromise((resolve, reject) => {
    console.log(2);
    setTimeout(() => {
        console.log('A', promise1.PromiseState);
        resolve('这次一定');
        console.log('B', promise1.PromiseState);
        console.log(4);
    });
})
promise1.then(
    result => {
        console.log('C', promise1.PromiseState);
        console.log('fulfilled:', result);
    },
    reason => {
        console.log('rejected:', reason)
    }
)
console.log(3);
```
输出结果：
``` js
1
2
3
A pending
C fulfilled
fulfilled: 这次一定
B fulfilled
4
```

**从上面的结果我们可以看到 `fulfilled: 这次一定` 打印出来啦，`promise1.then()`方法也正常执行，打印出了当前的状态：`B fulfilled`， 但是！！**

代码输出顺序还是不太对，原生Promise中，`fulfilled: 这次一定` 是最后输出的

这里有一个很多人忽略的小细节，**`resolve` 和 `reject` 是要在 `事件循环末尾` 执行的**，因此我们就 **给 `resolve` 和 `reject` 里面加上 `setTimeout`** 就可以了

**我们在判断完 promise 状态后再加 setTimeout ：**
``` js
class myPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
        this.onFulfilledCallbacks = []; // 保存成功回调
        this.onRejectedCallbacks = []; // 保存失败回调
        try {
            func(this.resolve.bind(this), this.reject.bind(this));
        } catch (error) {
            this.reject(error)
        }
    }
    resolve(result) {
        if (this.PromiseState === myPromise.PENDING) {
+           setTimeout(() => {
                this.PromiseState = myPromise.FULFILLED;
                this.PromiseResult = result;
                this.onFulfilledCallbacks.forEach(callback => {
                    callback(result)
                })
+           });
        }
    }
    reject(reason) {
        if (this.PromiseState === myPromise.PENDING) {
+           setTimeout(() => {
                this.PromiseState = myPromise.REJECTED;
                this.PromiseResult = reason;
                this.onRejectedCallbacks.forEach(callback => {
                    callback(reason)
                })
+           });
        }
    }
}
```
再测试一下：
``` js
1
2
3
A pending
B pending
4
C fulfilled
fulfilled: 这次一定
```
**可以看到最后输出 `fulfilled: 这次一定` ，和原生Promise顺序一致！**

到这里我们已经完成了 **promise的回调保存**，已经越来越接近胜利了:smiley_cat:

### 3.验证 then 方法多次调用
--------------
Promise 的 then 方法可以被多次调用

用一个 :chestnut:，来验证一下我们写的promise `then` 方法是否可以多次调用：
``` js
class myPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.PromiseState = myPromise.PENDING;
        this.PromiseResult = null;
        this.onFulfilledCallbacks = []; // 保存成功回调
        this.onRejectedCallbacks = []; // 保存失败回调
        try {
            func(this.resolve.bind(this), this.reject.bind(this));
        } catch (error) {
            this.reject(error)
        }
    }
    resolve(result) {
        if (this.PromiseState === myPromise.PENDING) {
            setTimeout(() => {
                this.PromiseState = myPromise.FULFILLED;
                this.PromiseResult = result;
                this.onFulfilledCallbacks.forEach(callback => {
                    callback(result)
                })
            });
        }
    }
    reject(reason) {
        if (this.PromiseState === myPromise.PENDING) {
            setTimeout(() => {
                this.PromiseState = myPromise.REJECTED;
                this.PromiseResult = reason;
                this.onRejectedCallbacks.forEach(callback => {
                    callback(reason)
                })
            });
        }
    }
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => {
            throw reason;
        };
        if (this.PromiseState === myPromise.PENDING) {
            this.onFulfilledCallbacks.push(onFulfilled);
            this.onRejectedCallbacks.push(onRejected);
        }
        if (this.PromiseState === myPromise.FULFILLED) {
            setTimeout(() => {
                onFulfilled(this.PromiseResult);
            });
        }
        if (this.PromiseState === myPromise.REJECTED) {
            setTimeout(() => {
                onRejected(this.PromiseResult);
            });
        }
    }
}


// 测试代码
const promise = new myPromise((resolve, reject) => {
    setTimeout(() => {
        resolve('success')
    }, 2000);
})
promise.then(value => {
    console.log(1)
    console.log('resolve', value)
})
promise.then(value => {
    console.log(2)
    console.log('resolve', value)
})
promise.then(value => {
    console.log(3)
    console.log('resolve', value)
})
```

运行上面 :chestnut:，输出结果：
``` js
1
resolve success
2
resolve success
3
resolve success
```

所有 `then` 中的回调函数都已经执行 :sunglasses:

说明我们当前的代码，已经可以实现 `then` 方法的多次调用:star2:

:clap::clap::clap: 完美，继续.....