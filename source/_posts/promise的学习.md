---
title: promise的学习
date: 2020-11-16 09:11:02
tags:
- js
- ES6
category: ES6系列
img: 2020/11/02/vue3/vue.png
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

这篇文章是阅读完[promiseA+](https://promisesaplus.com/)规范和[ES6入门教程](https://es6.ruanyifeng.com/)，写的总结。

### Promise对象

Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。

#### 术语

1. promise : 是拥有then方法的对象或者函数

2. thenable : 定义了then方法的对象或者函数

3. value : 任何JS的合法值

4. exception : 使用throw抛出的一个值

5. reason : promise被拒绝的原因

#### Promise的状态

1. pending (等待) ：promise初始化状态

2. fulfilled (执行) : pending => resolve

3. rejected （拒绝）: pending => rejected

#### Promise方法

promise必须提供一个```then```的方法，来访问当前的最终值或者原因

```
promise.then(onFulfilled,onRejected)
```

1. ```onFulfilled```和```onRejected```都是可选的参数，
如果有个不是函数的话，这个将会被忽略
    
    案例：
    ```
    const promise = new Promise((resolve,reject) => {
        if(/**异步执行成功**/){
            resolve(value)
        } else {
            resolve(error)
        }
    })


    function timeout(ms) {
        return new Promise((resolve, reject) => {
            setTimeout(resolve, ms, 'done');
        });
    }

    timeout(100).then((value) => {
        console.log(value);
    });
    // 输出的结果为：
    // Promise
    // done
    ```
    上述代码中，timeout返回一个Promise实例，表示过一段时间以后才会发生的结果。
    过了指定的时间(ms)后，Promise的状态变为resolved,然后触发```.then```的方法绑定的回调函数

    ```
    let promise = new Promise((resolve,reject) => {
        console.log('promise')
        resolve()
    })
    promise.then(() => {
        console.log('resolved')
    })
    console.log('Hi')
    // 输出结果：
    // ‘promise’
    // ‘Hi’
    // ‘resolved’
    ```
    这个例子中初始化后的Promise对象会立即执行，所以会先输出Promise，然后```then```方法制定的
    回调函数将在当前脚本中所有同步函数执行完后再去执行。所以，```resolved```最后输出

    ```
    let promise = new Promise(resolve => {
        console.log('promise')
        resolve()
    }).then(value => console.log('成功'))
    console.log('here')
    // 输出结果：
    // 'promise'
    // 'here'
    // 成功
    ```
    promise创建的过程是在js的同步任务中，所以会先输出promise，然后将resolve()放在微任务里。然后JS会继续执行剩下的同步任务，
    所以接下来会输出‘here’，最后当同步的任务执行完之后，开始执行微任务中的函数，输出‘成功’

    ```
    setTimeout(()=>{
        console.log('setTimeout')
    },0)
    let promise = new Promise(resolve => {
        console.log('promise')
        resolve()
    }).then(value => console.log('成功'))
    console.log('bottom')
    // 输出结果
    // promise
    // bottom
    // 成功
    // setTimeout
    ```
    代码执行中遇到setTimeout,会将setTimeout放到宏任务中，然后继续执行JS同步任务中的函数，首先是Promise的初始化，输出promise，
    然后遇到resolve后，会将resolve中的函数放在微任务中，继续执行同步任务中未执行的。输出bottom后，再输出成功，微任务执行完成后，会开始执行
    宏任务，输出setTimeout

    ```
    let promise = new Promise(resolve => {
       setTimeout(() => {
           console.log('setTimeout')
           resolve()
       },0)
       console.log('promise')
    }).then(value => console.log('成功'))
    console.log('bottom')
    // 输出结果
    // promise
    // bottom
    // setTimeout
    // 成功
    ```
    这个例子中首先JS会定义Promise，然后开始执行，发现setTimeout会将setTimeout放在宏任务中，然后继续执行，
    输出promise，此时还有同步任务没有执行完，所以会继续执行console.log('bottom')，输出bottom，执行完
    这步之后，JS同步任务已经执行完成了，接下来开始执行刚才放在宏任务中的setTimeout，先输出‘setTimeout’后，
    发现resolve，将其放在微任务中，如果此时setTimeout中还有其他的代码，会先执行setTimeout中的其他代码，
    再执行微任务中，输出‘成功’


2. Promise.prototype.then()

   .then方法是定义在原型对象Promise.prototype上的。它的作用是为Promise添加回调函数。
   第一个参数是resolve状态的回调函数，第二个参数是reject状态下的回调函数then返回一个**新的**```Promise```实例,后面的then是对前面Promise的处理

    ```
    new Promise((resolve,reject) => {
        resolve('买水去了')
    }).then(
        value => {
            console.log(value)
        },
        error => {
            console.log(error)
        }
    )
    ```

3. Promise.prototype.catch()

Promise.prototype.catch()的方法是.then(null,rejection) 或 .then(undefined,rejection)的别名，用于指定发生错误时的回调函数。

```
    new Promise((resolve,reject) => {
        reject(new Error('出现错误'))
    }).then(value => {
        console.log(value)
    }).catch(err) => {
        console.log(err) // 出现错误
    }

    const promise = new Promise((resolve,reject) => {
        resolve('ok')
        throw new Error('test Error')
    })
    promise.then(value => {console.log(value)})
        .catch(error => {console.log(error)}) 

    // 输出结果
    //ok
```
如果 Promise 状态已经变成resolved，再抛出错误是无效的。
Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。

4. Promise.prototype.finally()

finally()方法用于不管Promise对象最后的状态如何，都会执行该操作

```
promise.then(result => {

}).catch(error => {

}).finally(() => {
    
})

```
finally方法的回调函数不接受任何参数，这就说明没办法知道前面的Promise的状态是fulfilled还是
rejected.这就表明，在finally中的操作不依赖Promise的执行结果

5. Promise.all()

Promise.all()方法用于多个Promise实例，包装成一个新的promise实例

```
const p = Promise.all([p1,p2,p3])
p.then((res) => {
    // res是一个数组，是p1,p2,p3的返回值数组
}).catch((err) => {

})

const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
```
![输出结果](https://i.loli.net/2020/11/16/ZoQPcVENuqOiW4g.png)

上述代码中p1会resolved,p2首先会rejected,但是p2有自己的catch方法，该方法返回一个新的Promise
实例，p2的方法实际上指向这个实例。这个实例执行完catch的方法后，会变成resolveed，导致Promise
方法参数里的两个实例都会resolved，因此会调用then中的方法

如果p2中没有catch的方法，就会调用Promise.all()中的catch方法

6. Promise.race() 和Promise.all()方法一样，这里不再赘述

7. Promise.allSettled() -- ES2020 

Promise.allSettled()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只有等到所有这些参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。该方法由 ES2020 引入。

```
const p = [
    fetch('/api-1),
    fetch('/api-2),
    fetch('/api-3)
]
await Promise.allSettled(p)
removeLoadingIndicator()

```
上面代码对服务器发出三个请求，等到三个请求都结束，不管请求成功还是失败，加载的滚动图标就会消失。

该方法返回的新的 Promise 实例，一旦结束，状态总是fulfilled，不会变成rejected。
状态变成fulfilled后，Promise 的监听函数接收到的参数是一个数组，
每个成员对应一个传入Promise.allSettled()的 Promise 实例。

当我们只关心异步操作有没有结束的时候，就可以用Promise.allSettled()的方法。Promise.all()无法确定所有请求都结束。

8. Promise.any()
ES2021 引入了Promise.any()方法。该方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。
只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；
如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。

Promise.any()跟Promise.race()方法很像，只有一点不同，
就是不会因为某个 Promise 变成rejected状态而结束
