---
title: 2021.03 - 2021.04面总结
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





    