---
title: 2021.03 - 2021.04面试总结
date: 2020-12-09 11:30:09
tags:
- 前端面试题
- vue
category:
- 前端面试题
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

经历了一个月的准备加面试，收获也是蛮大的。今天把这一个月所经历的面试总结整理出来。这一个月面试了大厂和一些普通的公司。因为我没有react的项目以及对webpack的了解不深入，这两方面的面试题目面试官也没有深入去问。


#### JS相关
*这里每个知识点单拎出来都是一个知识体系，这里总结的只是在面试中的回答方式*
1. 什么是原型链？
   - 每个对象都有__proto__的属性，该属性的指向其实例对象中的原型对象的属性/方法，如果该实例对象找不到要查的属性/方法，就会向上查找该实例的原型对象的实例和方法
   - 构造函数的prototype属性指向其原型对象
   - 原型对象的constructor 属性指向构造函数

2. new这个关键词
   - 创建对象，并继承该构造函数
   - 执行该构造函数，并将this 指向新的对象实例
   - 返回新的值

3. JS实现new
    ```
        function myNew(foo,...args){
            let obj = Object.create(foo.prototype)
            let result = foo.apply(obj,args)
            return Object.prototype.toString.call(result) === '[object object]' ? return : obj
        }
    ```

4. 你了解的ES5的继承
    - 原型链继承
       **原型链继承的原理**：直接让子类的原型对象指向父类实例。当子类实例找不到对应的属性和方法时，就会往它的父类实例上查找，从而实现对父类的属性和方法的继承
       *代码示例*
        ```
        function Person(){
            this.name = ['Alice'] //如果这里只是基本类型，那么最不会出现缺点1的情况
        }
        // 父类原型的方法
        Parent.prototype.getName = function(){
            return this.name
        }
        // 子类
        function Child(){}
        
        // 让子类的原型对象指向父类实例，这样Child实例上找不到的方法和属性就会到原型对象上找
        Child.prototype = new Person()
        Child.prototype.constructor = Child
        // 然后Child实例就能访问到父类及其原型上的name属性和getName()方法
        const c1 = new Child()
        console.log(c1.name) // Alice
        console.log(c1.getName()) // Alice
        ```
       **原型链继承的缺点** 
       - 所有的Child实例原型都指向同一个Parent实例，因此对某个Child实例的父类引用类型变量修改会影响所有实例的Child实例
       - 创建子类实例时无法向父类构造传参，没有实现`super()`的功能
       ```
        const c2 = new Child()
        c1.name[0] = 'foo'
        console.log(c1.name) // ['foo']
        console.log(c2.name) // ['foo']
       ```
    - 构造函数继承
        构造函数的继承：是在子类的构造函数中执行父类的构造函数，并为其绑定子类的this，让父类的构造函数把成员属性和方法挂到子类的this上，这样就避免了多个实例之间共享一个原型实例，又能向父类的方法传参
        ```
        function Parent(name) {
            this.name = [name]
        }
        Parent.prototype.getName = function() {
            return this.name
        }
        function Child() {
            Parent.call(this, 'zhangsan')   // 执行父类构造方法并绑定子类的this, 使得父类中的属性能够赋到子类的this上
        }

        //测试
        const c1 = new Child()
        const c2 = new Child()
        c1.name[0] = 'foo'
        console.log(c1.name)          // ['foo']
        console.log(c2.name)          // ['zhangsan']
        c2.getName()  
        ```
        **构造函数继承**
        - 继承不到父类原型的属性和方法
    - 组合式继承
        原型链继承和构造函数继承组合
        ```
        function Parent(name) {
            this.name = [name]
        }
        Parent.prototype.getName = function() {
            return this.name
        }
        function Child() {
            // 构造函数继承
            Parent.call(this, 'Alice') 
        }
        //原型链继承
        Child.prototype = new Parent()
        Child.prototype.constructor = Child

        //测试
        const c1 = new Child()
        const c2 = new Child()
        c1.name[0] = 'foo'
        console.log(c1.name)          // ['foo']
        console.log(c2.name)          // ['Alice']
        c2.getName() 
        ```
        **组合式继承的缺点**
        - 每次创建子类实例都执行链两次构造函数(`Parent.call()` 和 `new Parent()`),在子类创建实例时，原型中会存在两份相同的属性和方法。代码不优雅
    - 寄生式组合继承
        针对组合式继承的缺点，将指向父类实例改成指向父类的原型。这样就减少了一次构造函数的执行
        ```
        function Parent(name) {
            this.name = [name]
        }
        Parent.prototype.getName = function() {
            return this.name
        }
        function Child() {
            // 构造函数继承
            Parent.call(this, 'Alice') 
        }
        //原型链继承
        Child.prototype = Object.create(Parent.prototype)  //将`指向父类实例`改为`指向父类原型`
        Child.prototype.constructor = Child

        //测试
        const c1 = new Child()
        const c2 = new Child()
        c1.name[0] = 'foo'
        console.log(c1.name)          // ['foo']
        console.log(c2.name)          // ['Alice']
        c2.getName()                  // ['Alice']
        ```





5. this指向
    - 在严格模式和非严格模式中this的指向

      非严格模式下，自执行函数的this指向window ；给元素的某个行为绑定一个方法，当行为触发时，执行绑定方法，此时this指向当前元素； 如果方法名前面有.，则this指向.前面的对象，如果没有，this默认指向window ； 在构造函数中，函数体中的this.xxx = xxx中的this,指的是当前类的实例 ； 使用call/apply/bind来改变this指向，传入的this是谁就指向谁

      严格模式下自执行函数的this指向undefined；执行方法时，如果方法前面有.，则this指向.前面的对象，如果没有，则指向undefined

      两者的区别 ： 对于JS代码中没有写执行主体的情况下，非严格模式默认时window执行的，所以this会默认执行window
      在严格模式下，没有写执行主体，this指向undefined

    - 普通函数、箭头函数和构造函数的this指向

      普通函数由于没有对象或者实例的调用，this指向window ; 构造函数的this指向调用它的实例(在严格模式下，this为undefined)
      箭头函数没有this，箭头函数中的this和函数或对象调用，它默认指向定义时所在上下文环境，逐级向上查找最近函数的作用域的this，箭头函数的this不会通过apply/call/bind发生改变  
    
    - node.js中的this指向

      全局中的this,默认是一个空对象。在全局中的this与global对象没有任何关系，指向的是modules.exports

      函数中的this，指向global对象。在函数中通过this定义的变量，都会挂载到global对象上

      构造函数的this指向它的实例



6. 防抖和节流

   - 防抖：在时间被触发n秒后再执行回调。如果在着n秒内多次被触发，则重新计时。
   - 使用场景： 在搜索时关键词的联想

   - 节流 ： 高频事件触发,n秒内只执行一次，如果单位时间内多次触发，只有一次生效。节流函数稀释函数执行的频率 
   - 使用场景： 提交表单、拖拽、缩放，防止高频触发位置变动

   - 区别：防抖是多次触发只执行最后一次，节流是多次触发变成每隔一段时间执行一次


7. 深拷贝和浅拷贝
    - 深拷贝：增加了一个指针，并申请了一个新的内存，新增的这个指针指向这个新的内存

    - 浅拷贝：只是增加了一个指针指向已存在的内存地址

    - 区别 是否可以完全复制一个对象

    - 代码实现
    ```
    // 不推荐
    function deepClone(obj){
        let result = JSON.stringify(obj)
        return JSON.parse(result)
    }
    function deepClone(obj){
        let newObj = obj instanceof Array ? [] :{}
        if(typeof obj !== 'object'){}
    }
    ```

8. Event Loop 事件循环机制

    - JS相关知识：
        1. JS作为浏览器的脚本语言，主要是与用户交互及操作DOM。因此是单线程的，这也避免了同时操作同一个DOM的矛盾问题 
        2. 利用多核CPU的计算，H5的web worker 实现了“多线程”，实际上指的是“多子线程”，完全受控于主线程，且不允许操作DOM
        3. JS引擎在monitoring process进程，会持续不断的进行检测主线程执行栈是否为空，一旦为空，就回去Event Queue那里检查是否有等待被调用的函数。这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）
        4. 所有同步任务都在主线程上执行，形成一个执行栈
        5. 如果在微任务执行期间微任务的队列加入了新的微任务，会将新的微任务放在队列的尾部，之后也会被执行

    - JS中异步操作
        `setTimeout` `setInterval` `ajax` `promise` `async await` `I/O`

    - 同步任务：在主线程中排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务
    - 异步任务：不进入主线程，而进入任务队列的任务，只有任务队列通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行

    - 宏任务(macro-task) : 整体代码:script,setTimeout , setInterval I/O Rendering setImediate
    - 微任务(mincro-task) : promise.then(),process.nextTick(node) mutationObserver

    - [事件循环机制](https://upload-images.jianshu.io/upload_images/4820992-82913323252fde95.png)：
    
       1. 整段script标签在开始执行的时候，会把所有的代码分为两部分：同步任务和异步任务
       2. 同步任务会直接进入到主线程中执行 
       3. 异步任务又分为微任务和宏任务
       4. 宏任务进入到Event Table 中，并在里面注册回调函数，每当指定的事件完成后，Event Table 会将这个函数移到Event Queue中
       5. 微任务也会进入到另一个Event Table中，并在里面注册回调函数，每当指定的事件完成后，Event Table 会将这个函数移到Event Queue中
       6. 当主线程内的任务执行完毕，主线程为空时，会检查微任务的Event Queue ，如果有任务，就全部执行，没有的话，就执行下一个宏任务
       7. 上述过程会不断重复，这就是Event loop事件循环
9. 事件委托(事件代理)概念及使用场景
    - 什么是事件委托：
    把元素响应事件的函数委托到其父层或者更外层元素上，当事件响应到需要绑定的元素上，利用事件冒泡的机制触发它的外层元素的事件上，然后在外层元素上执行函数
    
    - 事件模型的三个阶段：捕获、目标、冒泡
    
    - 事件委托的优点：减少内存的消耗 ； 动态绑定事件


10. 闭包的理解
    - 什么是闭包？
        函数A中返回了一个函数B，并且在B中使用了函数A中的变量，函数B被成为闭包

    - 闭包的原理

       - 函数执行分为两个阶段：预编译阶段和执行阶段
        在预编译阶段如果发现内部函数使用了外部函数的变量，则在内存中创建一个“闭包”对象并保存对应变量的值，如果已存在“闭包”，则只需要增加对应属性值即可；
        执行完后，函数执行上下文会被销毁，函数对“闭包”对象的引用也会销毁，但其内部函数还持用该“闭包”的引用，所以内部函数可以继续使用外部函数的变量

       - 利用函数作用域的特性，一个函数内部定义的函数会将包含外部函数的活动对象添加到它的作用域链中，函数执行完后，其执行作用域链仍在引用这个活动对象，所以其活动对象不会被销毁，直到内部函数被销毁后才被销毁

    - 闭包的特点
      - 函数嵌套函数
      - 函数内部可以引用函数外部的参数和变量
      - 参数和变量不会被垃圾回收机制回收 

    - 闭包的优缺点
        - 优点
            保护函数内部变量的安全，实现封装，防止变量流入其他环境发生命名冲突
            在内存中维持一个变量，可以作为缓存
            匿名自执行函数可减少内存消耗

        - 缺点
            不会被垃圾回收机制回收，需要手动设置为null，否则会造成内存泄露
            对处理速度有负面影响，闭包的层级决定了引用的外部变量，在查找时经过的作用域链长度
        
    - 使用场景
        模块封装
        在循环中创建闭包，防止取到意外的值

11. Promise的理解

    - 什么是Promise

        Promise 简单来说是一个容器，里面包含着某个未来才会结束的事件的结果。Promise是一个对象，从它可以获取异步操作的消息。Promise提供统一的API

    - Promise的特点

        1. 对象不受外界的影响，有三种状态，`pending`,`fulfilled`,`rejected`
        2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。从`pending`变成`fulfilled`和`pending`变成 `rejected`

    - Promise的优点

        可以将异步操作以同步操作的流程表达出来，避免了回调地狱
        Promise对象提供了统一的接口，使控制异步操作更加容易

    - Promise的缺点
    - Promise的常用API