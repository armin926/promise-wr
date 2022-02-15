<!--
 * @Descripttion: 实现 resolve 和 reject
 * @Author: armin
 * @Date: 2022-02-15 14:32:36
 * @LastEditors: armin
 * @LastEditTime: 2022-02-15 14:39:38
-->
实现 resolve 和 reject
-----------------------
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