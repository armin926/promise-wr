<!--
 * @Descripttion: 回顾Promise知识点
 * @Author: armin
 * @Date: 2022-02-14 17:14:45
 * @LastEditors: armin
 * @LastEditTime: 2022-02-14 17:14:46
-->
手写Promise（1）——定义初始结构，实现resolve和reject
====================================================
Promise核心要点
---------------
Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）
一个 Promise 必然处于以下几种状态之一 👇：

- 待定 (pending): 初始状态，既没有被兑现，也没有被拒绝。
- 已成功 (fulfilled): 意味着操作成功完成。
- 已拒绝 (rejected): 意味着操作失败。

当 promise 被调用后，它会以**处理中状态** (pending) 开始。 这意味着调用的函数会继续执行，而 promise 仍处于处理中直到解决为止，从而为调用的函数提供所请求的任何数据。
被创建的 promise 最终会以**被解决状态** (fulfilled) 或 **被拒绝状态** (rejected) 结束，并在完成时调用相应的回调函数（传给 **then** 和 **catch**）。

示例：

    let p1 = new Promise((resolve, reject) => {
        resolve('成功')
        reject('失败')
    })
    console.log('p1', p1)

    let p2 = new Promise((resolve, reject) => {
        reject('失败')
        resolve('成功')
    })
    console.log('p2', p2)

    let p3 = new Promise((resolve, reject) => {
        throw('报错')
    })
    console.log('p3', p3)

这里包含了四个知识点 👇：

- 1、执行了resolve()，Promise状态会变成fulfilled，即 **已完成状态**
- 2、执行了reject()，Promise状态会变成rejected，即 **被拒绝状态**
- 3、Promise只以第一次为准，第一次成功就永久为fulfilled，第一次失败就永远状态为rejected
- 4、Promise中有throw的话，就相当于执行了reject()

接下来看下面一段代码，学习新的知识点：

    let myPromise1 = new Promise(() => {});

    console.log('myPromise1 :>> ', myPromise1);

    let myPromise2 = new Promise((resolve, reject) => {
        let a = 1;
        for (let index = 0; index < 5; index++) {
            a++;
        }
    })

    console.log('myPromise2 :>> ', myPromise2)

    myPromise2.then(() => {
        console.log("myPromise2执行了then");
    })

这里包含了三个知识点 👇：

- 1、Promise的初始状态是pending
- 2、Promise里没有执行resolve()、reject()以及throw的话，这个**promise的状态也是**pending
- 3、基于上一条，pending状态下的promise不会执行回调函数then()

最后一点：

    let myPromise0 = new Promise();
    console.log('myPromise0 :>> ', myPromise0);
    
这个里包含了一个知识点：
- 规定必须给Promise对象传入一个执行函数，否则将会报错。