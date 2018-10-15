---
title: 轻松搞定javascript中this指向
date: 2018-10-15 10:45:18
tags: javascript
---

　　关于javascript中this指向的问题，现总结如下，如有不正确，欢迎指正。
    
　　javascript中，this的指向并不是在函数定义的时候确定的，而是在其被调用的时候确定的。也就是说，`函数的调用方式决定了this指向`。记住：`this` 就是一个指针，指向我们调用函数的对象。

　　在此将javascript中this的调用方式分为以下几种：

### 1、直接调用：
　　直接调用是指通过 funName(..) 这种方式调用。此时，函数内部的this指向全局变量。
```js
function foo() {
    console.log(this === global);
}
foo(); //true
```

　　注意：直接调用并不是仅仅指在全局作用域下进行调用，而是说在任何作用域下通过funName(..) 这种方式调用。如下两个例子亦是属于直接调用方式：

　　a、使用bind函数改变外层函数的作用域，然后在内层直接调用，其this指向依然是全局变量：

```
function foo() {
    console.log(this === global);
}

function foo1() {
    foo();
};
var foo2 = foo1.bind({}); //改变foo1函数的作用域
foo2(); //true 依然指向全局变量
```

　　b、函数表达式，将一个函数赋值给一个变量，然后通过直接调用的调用方式调用此函数，其this指向依然是全局变量：

```js
var obj = {
    foo: function(){
        console.log(this === obj);   //false
        console.log(this === global);//true
    }
}
var f = obj.foo;
f();
```
### 2、方法调用：

　　方法调用是指通过对象来调用其方法函数，类似于obj.foo(..)的调用方式。此时，this指向调用该方法的对象，注意，是最终调用该方法的对象。
```js
var obj = {
    foo1: function(){
        console.log(this === obj);
    }
}
obj.foo2 = function() {
    console.log(this === obj);
}
function foo3() {
    console.log(this === obj);
}
obj.foo3 = foo3;
obj.foo1(); // true
obj.foo2(); // true
obj.foo3(); // true
```
### 3、new 调用：

　　在es5中，通过 new Constructor() 的形式调用一个构造函数，会创建这个构造函数实例，而这个实例的this指向创建的这个实例。如下例所示，在构造函数内部使用this.name并没有改变全局变量name的值。

```js
var name = "全局";
function Person(name){
    this.name = name;
}
var p = new Person("局部");
console.log(p.name);
console.log(name);
```

上述三种情况是常见的调用方式，以下还有一些特殊的调用方式：如bind、call、apply以及es6中箭头函数。

### 4、bind函数对this的影响

　　bind函数用于绑定this的指向，并且返回一个绑定函数，此函数绑定了this指向，注意：绑定函数的this指向将不可再次更改。

```js
var obj = {};
function foo1() {
    console.log(this === obj);
}
foo1(); //false  此时属于上述直接调用的方式，因此其this指向global
var foo2 = foo1.bind(obj);
foo2(); //true 绑定函数的this指向其绑定的对象
/**
 * 注意：绑定函数的this指向不可更改
 */
var foo3 = foo2.bind({'a': 1});
foo3(); //true 
foo2.apply({'a': 1}); //true 
foo2.call({'a': 1});  //true 
```

###  5、apply和call对this指向的影响

　　apply和call亦可以用于改变this指向，并且返回执行结果。关于bind、apply和call的区别以及实现可以参考我的博客：[《bind、call、apply的区别与实现》][2]，在此不做重复说明。需要注意的一点是：apply和call不可以改变绑定函数(使用bind返回的函数)的this指向。

```js
var obj = {};
function foo1() {
    console.log(this === obj);
}
var foo2 = foo1.bind({'a': 1});
/**
 * 注意：此处并不是绑定函数，因此其返回值依然是ture，即apply改变其this指向。
 */
foo1.apply(obj); //true
/**
 * 注意：此处是绑定函数，因此不可再通过apply和call的形式改变其this指向。
 */
foo2.apply(obj); //false
```
### 6、es6箭头函数中的this

　　箭头函数没有自己的this绑定，其使用的this是其直接父级函数的this。也就是说，箭头函数内部的this是由其直接外层函数（方法）决定的，而外层函数中的this是由其调用方式决定的。

```js
const obj = {
    foo: function() {
        const inner = () => {
            console.log(this === obj);
        };
        inner();
    },
    far: function() {
        return () => {
            console.log(this === obj);
        }
    }
}
/**
 * inner()内的this是foo的this，其指向取决于foo的调用方式
 */
obj.foo(); //true
var foo1 = obj.foo;
foo1();    //false 此时应该指向global

const far1 = obj.far();
far1();    //true
const far2 = obj.far;
far2()();  //false 此时应该指向global
```

　　以上是个人对JavaScript中this的理解，欢迎您的留言。

参考文献：
1、《[JavaScript 的 this 指向问题深度解析][1]》


  [1]: https://segmentfault.com/a/1190000008400124
  [2]: https://www.cnblogs.com/chengbo-x/p/9735677.html
  [3]: https://www.cnblogs.com/chengbo-x/p/9744911.html
