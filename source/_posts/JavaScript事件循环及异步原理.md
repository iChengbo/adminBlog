---
title: JavaScript事件循环及异步原理
date: 2018-10-16 16:01:34
tags: [JavaScript,事件循环]
categories: 前端
---

## JS的语言特点：
---
1. JS是单线程的，只有一个主线程；
2. 函数内的代码是从上到下顺序执行的，遇到函数调用则执行被调用函数，待完成后继续执行；
3. 遇到异步事件，`宿主环境`另开一个线程，主线程继续执行，待异步函数结果返回后，执行回调函数。

其中，`宿主环境`指`浏览器`和`NodeJs`:

+ 在浏览器中：浏览器负责提供 这个额外的线程；
+ 在NodeJs中：`NodeJs`借助`libuv`来作为抽象封装层，从而屏蔽不同操作系统的差异，`NodeJs`可以借助`libuv`来实现多线程。

## 为什么JavaScript是单线程？

&nbsp;&nbsp; `JavaScript` 语言的一大特点就是单线程，也就是说，同一个时间只能做一件事。这样设计的方案主要源于其语言特性，因为 `JavaScript` 是浏览器脚本语言，它可以操纵 `DOM` ，可以渲染动画，可以与用户进行互动，如果是多线程的话，执行顺序无法预知，而且操作以哪个线程为准也是个难题。</br>
&nbsp;&nbsp; 所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。</br>
&nbsp;&nbsp; 在 `HTML5` 时代，浏览器为了充分发挥 `CPU` 性能优势，允许 `JavaScript` 创建多个线程(`Web Worker`)，但是即使能额外创建线程，这些子线程仍然是受到主线程控制，而且不得操作 `DOM`，类似于开辟一个线程来运算复杂性任务，运算好了通知主线程运算完毕，结果给你，这类似于异步的处理方式，所以本质上并没有改变 `JavaScript` 单线程的本质。

## 函数调用栈与任务队列
### 函数调用栈
&nbsp;&nbsp; `JavaScript` 只有一个主线程和一个调用栈（`call stack`），那什么是调用栈呢？</br>
(闲下来时再更新此处)

### 任务队列
&nbsp;&nbsp; 所有任务可以分成两种，一种是 `同步任务（synchronous）`，另一种是 `异步任务（asynchronous）`。

&nbsp;&nbsp; `同步任务`指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。</br>
&nbsp;&nbsp; `异步任务`指的是，不进入主线程、而进入`"任务队列"（task queue）`的任务，只有 `"任务队列"` 通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。</br>

&nbsp;&nbsp; 所以，当在执行过程中遇到一些类似于 `setTimeout` 等异步操作的时候，会交给浏览器的其他模块(以webkit为例，是webcore模块)进行处理，当到达 `setTimeout` 指定的延时执行的时间之后，回调函数会放入到任务队列之中。一般不同的异步任务的回调函数会放入不同的任务队列之中。等到调用栈中所有任务执行完毕之后，接着去执行任务队列之中的回调函数。
> 用Philip Roberts的演讲《Help, I’m stuck in an event-loop》之中的一张图表示就是

![事件循环栈](/事件循环栈.png)

在上图中，调用栈中遇到`DOM操作`、`ajax请求`以及`setTimeout`等WebAPIs的时候就会交给浏览器内核的其他模块进行处理，webkit内核在Javasctipt执行引擎之外，有一个重要的模块是webcore模块。对于图中WebAPIs提到的三种API，`webcore`分别提供了`DOM Binding`、`network`、`timer`模块来处理底层实现。等到这些模块处理完这些操作的时候将回调函数放入任务队列中，之后等栈中的`task`执行完之后再去执行任务队列之中的回调函数。

**... 此部分示例空闲时再补**


上面的流程解释了浏览器遇到 `setTimeout` 之后究竟如何执行的，其实总结下来就是以下几点：

1. 调用栈顺序调用任务;
2. 当调用栈发现异步任务时，将异步任务交给其他模块处理，自己继续进行下面的调用;
3. 异步执行完毕，异步模块将任务推入任务队列，并通知调用栈;
4. 调用栈在执行完当前任务后，将执行任务队列里的任务;
5. 调用栈执行完任务队列里的任务之后，继续执行其他任务;
这一整个流程就叫做 `事件循环（Event Loop）`。





## 宏任务和微任务
&nbsp;&nbsp; 任务队列又分为 `macro-task（宏任务）` 与 `micro-task（微任务）` ，在最新标准中，它们被分别称为 `task` 与 `jobs`。
+ macro-task（宏任务）大概包括：script(整体代码), setTimeout, setInterval, setImmediate（NodeJs）, I/O, UI rendering。
+ micro-task（微任务）大概包括: process.nextTick（NodeJs）, Promise, Object.observe(已废弃), MutationObserver(html5新特性)。

&nbsp;&nbsp; 事实上，事件循环决定了代码的执行顺序，从全局上下文进入函数调用栈开始，直到调用栈清空，然后执行所有的 `micro-task（微任务）`，当所有的 `micro-task（微任务）`执行完毕之后，再执行 `macro-task（宏任务）`，其中一个 `macro-task（宏任务）`的任务队列执行完毕（例如setTimeout 队列），再次执行所有的 `micro-task（微任务）`，一直循环直至执行完毕。

**示例待补**

## 总结
1. 不同的任务会放进不同的任务队列之中。
2. 先执行 `macro-task`，等到函数调用栈清空之后再执行所有在队列之中的 `micro-task`。
3. 等到所有 `micro-task` 执行完之后再从 `macro-task`中的一个任务队列开始执行，就这样一直循环。
4. 宏任务和微任务的大致划分如下：
    + macro-task（宏任务）：script(整体代码), setTimeout, setInterval, setImmediate（NodeJs）, I/O, UI rendering。
    + micro-task（微任务）: process.nextTick（NodeJs）, Promise, Object.observe(已废弃), MutationObserver(html5新特性)。


参考文章：

1. 《[JavaScript 事件循环及异步原理（完全指北）][1]》
2. 《[细说JavaScript事件循环机制-第一讲][2]》

[1]: https://www.cnblogs.com/liangyin/p/9783342.html
[2]: http://www.php.cn/js-tutorial-387676.html