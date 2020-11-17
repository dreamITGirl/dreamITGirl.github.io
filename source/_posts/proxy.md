---
title: proxy的理解
date: 2020-11-03 09:33:21
cover: true
tags:
- js
- ES6
category: ES6系列
img: 2020/11/16/promise-de-xue-xi/es6.jpeg
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---

欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

### Proxy

#### 概述

- proxy用于修改某些操作的默认行为，等同于语言层面的修改，属于一种”元编程“。

- proxy可以理解为，在目标对象之前设置了一层拦截，外界访问这个对象时，都要先通过这层拦截。因此提供了一种机制，可以对外界的访问进行过滤和改写。像Vue中的computed属性和双向绑定原理中都是和proxy同样的原理。

- 语法
```
const p = new Proxy(target,handler)
```
- target：要使用proxy包装的目标对象（可以是任何类型的对象，包括原生数组、函数，甚至另一个代理）
- handler:一个通常以函数作为属性的对象，各属性的函数分别定义了在执行各种操作时代理p的行为

- 举个例子
```
var handler = {
    get:function(target,name){
        console.log(target,name)
        return 66
    }
}
var proxy = new Proxy({},handler)

console.log(proxy.name)
console.log(proxy.time)
console.log(proxy.title)
```
![输出结果](https://i.loli.net/2020/11/17/4Rxs8CnJUaPfBzH.png)

```
var handler = {
    get:function(target,name){
        console.log('get方法')
        return 77
    },
    set:function(target,name,value){
        console.log('set方法')
        target[name] = value
    }
}
var proxy = new Proxy({},handler)
console.log(proxy.name)
console.log(proxy.time)
proxy.title = '测试代码'
```
![输出结果](https://i.loli.net/2020/11/17/7ihaTAEXMDQ43zg.png)

- 一个技巧是将 Proxy 对象，设置到object.proxy属性，从而可以在object对象上调用；Proxy 实例也可以作为其他对象的原型对象；
同一个拦截函数可以设置多个拦截操作

- 再举个例子
```
    var handler = {
        get:function(target,name){
            if(name == 'prototype'){
                return Object.prototype
            }
            return 'Hello' + name
        },
        apply:function(target,thisBinding,args){
            console.log(thisBinding) // 被调用时的上下文对象。
            return args[0]
        },
        construct:function(target,args){
            return {value:args[1]}
        }
    }
    var fproxy = new Proxy(function(x,y) {
        return x+y
    },handler)

    fproxy(1, 2) // 1
    new fproxy(1, 2) // {value:2}
    fproxy.prototype === Object.prototype // true
    fproxy.foo === "Hello, foo" // true
```
- Proxy支持的拦截操作：

    - get(target,property,receiver) : 拦截对象属性的读取

    - set(target,property,value,receiver):拦截对象属性的设置，返回一个布尔值

    - has(target,property): 拦截```property in proxy```的操作，返回一个布尔值

    - ownKeys(target):拦截```Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in```循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。

    - getOwnPropertyDescriptor(target, propKey)：拦截```Object.getOwnPropertyDescriptor(proxy, propKey)```，返回属性的描述对象。

    - defineProperty(target, propKey, propDesc)：拦截```Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)```，返回一个布尔值。

    - preventExtensions(target)：拦截```Object.preventExtensions(proxy)```，返回一个布尔值。

    - getPrototypeOf(target)：拦截```Object.getPrototypeOf(proxy)```，返回一个对象

    - isExtensible(target)：拦截```Object.isExtensible(proxy)```，返回一个布尔值

    - setPrototypeOf(target, proto)：拦截```Object.setPrototypeOf(proxy, proto)```，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截

    - apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如```proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)```。

    - construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new ```proxy(...args)```。





#### 实例方法

1. **```get()```**
    该方法会拦截目标对象的以下操作
    - 访问属性：```proxy[foo]和proxy.bar```
    - 访问原型链上的属性：```Object.create(proxy)[foo]```
    - ```reflect.get()```

    以下情况，proxy会抛出```TypeError```
    - 如果访问的目标属性是不可写以及不可配置的，则返回的值必须与该目标属性的值相同
    - 如果要访问的目标属性没有配置访问的方法，即get方法是undefined，则返回值必须是undefined

   *举个例子*

   ```
   var person = {
       name:'Alice'
   },
   handler = {
       get:function(target,propkey){
           if(propkey in target){
               return target[propkey]
           } else {
               throw new ReferenceError(`prop name ${propkey} does not exist.`)
           }
       }
   }

   var proxy = new Proxy(person,handler)

   console.log(proxy.name) // Alice
   console.log(proxy.age) // ReferenceError
   ```
    **get方法可以继承**
    ```
     let proto = new Proxy({},{
       get:function(target,name,receiver){
           console.log('get'+name)
           return target[name]
       }
   })
   let obj = Object.create(proto)
   console.log(obj.foo) // getfoo
    ```

2. **```set()```**
   该方法会拦截目标对象的以下操作

   - 指定属性值 ```proxy[foo] = bar``` 和 ```proxy.foo = bar```  
   - 指定继承者的属性值：```Object.create(proxy)[foo] = bar ```
   - reflect.set()

   以下的情况，proxy 会抛出一个 TypeError 异常：
    - 若目标属性是一个不可写及不可配置的数据属性，则不能改变它的值。
    - 如果目标属性没有配置存储方法，即 [[Set]] 属性的是 undefined，则不能设置它的值。
    - 在严格模式下，如果 set() 方法返回 false，那么也会抛出一个 TypeError 异常

    *举个例子*
    ```
    let p = new Proxy({},{
        set:function(target,prop,value,receiver){
            target[prop] = value
            console.log(`property set ${prop} = ${value}`)
            return true
        }
    })
    console.log('name' in p)  // false
    p.name = 10
    console.log('name' in p) // true
    console.log(p.name) // 10
    ```
    ![输出结果](https://i.loli.net/2020/11/17/sIZLMRkuTlVY4Eg.png)

    ```
        const handler = {
            set:function(target,prop,value,receiver){
                target[prop] = receiver
            }
        }
        const proxy = new Proxy({},handler)
        proxy.foo = 'bar'
        proxy,foo === proxy
    ```
    
    ![输出结果](https://i.loli.net/2020/11/17/SUxViMyCZwPoDbz.png)

    ```set```方法的第四个参数```receiver```，一般情况下```proxy```实例本身
    ```
    const handler = {
        set: function(obj, prop, value, receiver) {
            obj[prop] = receiver;
        }
    };
    const proxy = new Proxy({}, handler);
    const myObj = {};
    Object.setPrototypeOf(myObj, proxy);

    myObj.foo = 'bar';
    myObj.foo === myObj
    ```

3. **```apply()```**
    该方法会拦截目标对象的以下操作：
    - proxy(...args)
    - ```Function.prototype.apply()``` 和 ```Function.prototype.call()```
    - ```Reflect.apply()```

    以下情况，代理将抛出一个TypeError：

    - target必须是可被调用的。也就是说，它必须是一个函数对象。

    *举个例子*
    ```
    var p = new Proxy(function() {}, {
                apply: function(target, thisArg, argumentsList) {
                    console.log('called: ' + argumentsList.join(', '));
                    return argumentsList[0] + argumentsList[1] + argumentsList[2];
                }
            });
    console.log(p(1,2,3))
    ```
4. **```construct()```**
    该方法用于拦截new操作符，为了使```new```操作符在生成的proxy对象上生效，用于初始化代理的目标对象自身必须具有[[```Construct```]]内部方法(也就是说```new target``` 必须是有效的)

    *举个例子*
    ```
    var p = new Proxy(function(){},{
        construct:function(target,args,newTarget){
            console.log(`called ${args.join(',')}`)
            return {value:args[0] * 10}
        }
    })
    console.log(new p(1).value)
    ```

    ![输出结果](https://i.loli.net/2020/11/17/jBsqTwRSZ7LJxkh.png)

#### this问题
> 虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 Proxy 代理的情况下，目标对象内部的this关键字会指向 Proxy 代理。

```
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```
上面的代码中，一旦proxy代理了target,targot内部的this的指向就会指向proxy,