<!--
 * @Descripttion: 手写Promise——定义初始结构结构
 * @Author: armin
 * @Date: 2022-02-14 17:17:03
 * @LastEditors: armin
 * @LastEditTime: 2022-02-15 14:40:05
-->

:gift_heart:定义初始结构
-------------
原生的promise我们一般都会用new来创建实例 :point_down: ： 
```js
    let promise = new Promise()
```
所以我们手写的时候可以用构造函数或者class来创建，为了方便代码的整体观看就用class。  
把我们手写的Promise命名为myPromise，首先创建一个myPromise类   
```js
    class myPromise {}
```
再new一个promise实例的时候肯定需要传入参数
```js
    let promise = new Promise(() => {})
```
我们知道，这个参数是一个函数，并且当我们传入这个函数参数的时候会自动执行，因此我们需要在类的构造函数里面添加一个函数   
```js
    class myPromise {
    +    constructor(func) {
    +        func()
    +    }
    }
```