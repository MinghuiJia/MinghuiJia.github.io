---
title: setTimeout异步
date: 2022-04-25 19:34:03
excerpt: setTimeout函数是JavaScript中典型的异步操作，本篇文章从setTimeout入手，加深对同步与异步区别的理解，并且理解JS引擎单线程如何实现异步
index_img: https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/setTimeout_async/async2.png
tags:
  - JavaScript
  - 异步
  - 线程
  - 闭包
categories:
  - JavaScript教程
---

# 前言
setTimeout函数是JavaScript中典型的异步操作，本篇文章从setTimeout入手，加深对同步与异步区别的理解，并且理解JS引擎单线程如何实现异步

# 同步与异步任务
## 定义
- 同步：请求发起方对消息结果的获取是**主动发起的**
- 异步：请求发起方对消息结果的获取是**被动通知的**

同步与异步在很多场景下会与阻塞与非阻塞搞混，因为在很多场合下感觉没什么区别
- 阻塞：一个函数被调用后，等待函数返回结果之前，当前线程处于**挂起状态**
- 非阻塞：一个函数被调用后，等待函数返回结果之前，当前线程处于**运行状态**

## 同步异步与阻塞非阻塞的区分方式
- 同步与异步的判别准则：获取消息结果的方式是**主动还是被动**
- 阻塞与非阻塞的判别准则：当前线程处于**挂起**还是**运行**状态（挂起：当前线程什么都不能干；运行：当前线程可以处理其他任务）

{% note info %}
排在异步任务后面的代码，不用等待异步任务结束就会运行，即异步任务不具备“阻塞”效果
{% endnote %}

## JavaScript单线程模式
JavaScript是单线程的，但是其运行环境（Chrome浏览器）是多线程的
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/setTimeout_async/async1.png)

### 浏览器多线程介绍
浏览器中的线程包括但不限于：
- GUI线程
GUI线程就是渲染页面的，他解析HTML和CSS，然后将他们构建成DOM树和渲染树

- JS引擎线程
这个线程就是负责执行JS的主线程，也是上述提到**JS是单线程**所指代的线程。**这个线程跟GUI线程是互斥的**，互斥的原因是JS也可以操作DOM，如果JS线程和GUI线程同时操作DOM，结果就混乱了，不知道到底渲染哪个结果

- 定时器线程
前面异步例子的`setTimeout`就运行在这里，他跟JS主线程不在同一个地方，所以“单线程的JS”能够实现异步。JS的定时器方法还有setInterval，也在这个线程

- 事件触发线程
定时器线程其实只是一个计时的作用，他并不会真正执行时间到了的回调，**真正执行这个回调的还是JS主线程**。当时间到了定时器线程会将这个回调事件给到事件触发线程，然后**事件触发线程将它加到任务队列里面去**。最终JS主线程从任务队列取出这个回调执行
	***注意：事件触发线程不仅仅只会将定时器事件放入任务队列，其他满足条件的事件也是由他负责放进任务队列***

- 异步HTTP请求线程
这个线程负责处理异步的ajax请求，当请求完成后，他也会通知事件触发线程，然后事件触发线程将这个事件放入任务队列给主线程执行
{% note warning %}
**所以JS异步的实现靠的就是浏览器的多线程，当他遇到异步API时，就将这个任务交给对应的线程，当这个异步API满足回调条件时，对应的线程又通过事件触发线程将这个事件放入任务队列，然后主线程从任务队列取出事件继续执行**
{% endnote %}

### 事件循环机制
事件循环是让JavaScript做到既单线程，又不会阻塞的核心机制
- 事件循环（Event Loop）
事件循环的作用：监控**调用栈**和**任务队列**，当调用栈为空，它就会取出任务队列中的一个回调函数，然后将它压入调用栈并执行
{% note info %}
事件循环并不是在ECMAScript标准中定义，而是在HTML标准中定义。即**属于JavaScript Runtime**而不属于JavaScript Engine
{% endnote %}

- 任务队列
任务队列分为：宏任务队列（鼠标、键盘事件、定时器相关的事件、AJAX等）和微任务队列（Promise）
宏任务队列的规则：
> 1. 来自相同任务源的任务必须放在同一个任务队列中
> 2. 来自不同任务源的任务可以放在不同任务队列中
> 3. 任务队列中的任务按顺序执行
> 4. 不同的任务队列，浏览器会进行调度，允许优先执行来自特定任务源的任务（鼠标、键盘事件被优先调用，保证流畅的用户体验）

- 任务队列执行过程
JavaScript运行时，除了一个正在运行的主线程，引擎还提供多个任务队列（根据任务的类型，所以有多个）
> 1. 首先，主线程会去执行所有的同步任务。等到同步任务全部执行完，就会去看任务队列里面有没有事件回调
> 2. 如果有，则取出一个回调事件重新进入主线程执行，这时它就变成同步任务了
> 3. 等到执行完，下一个异步任务再进入主线程开始执行，一旦任务队列清空，程序就结束执行
> 4. 只要同步任务执行完了，引擎就会一遍又一遍地去检查那些挂起来的异步任务，是不是可以进入主线程了。这种循环检查的机制，就叫做**事件循环（Event Loop）**
{% note warning %}
异步任务的写法通常是回调函数，一旦异步任务重新进入主线程，就会执行对应的回调函数。
{% endnote %}

### 宏任务队列与微任务队列关系
- 宏任务队列与微任务队列关系
![](https://gcore.jsdelivr.net/gh/MinghuiJia/CDN-source/setTimeout_async/async3.png)
事件循环的每一次循环成为`tick`，其任务细节为：
	- 调用栈选择最先进入队列的宏任务（通常是script整体代码），如果有则执行
	- 检查是否存在微任务，如果存在，则不断执行微任务，直至清空微任务队列
	- 浏览器更新渲染，每次事件循环，浏览器都可能完成更新渲染
- 任务队列执行先后顺序
{% codeblock %}
// 执行顺序问题，考察频率挺高
setTimeout(function() {
  console.log(1);
});
new Promise(function(resolve, reject) {
  console.log(2);
  resolve(3);
}).then(function(val) {
  console.log(val);
});
console.log(4);
{% endcodeblock %}
	1. 先执行同步代码
		- 执行`new Promise`中的`console.log(2)`，`then`后面的属于微任务，跳过
		- 然后执行`console.log(4)`
	2. 执行完`script`这个宏任务后，执行微任务(`Promise.then`)中的`console.log(val)`，此时`val`值由`resolve(3)`传递过来
	3. 执行另一个宏任务`setTimeout`中的`console.log(1)`


# setTimeout异步循环中的闭包

## 循环中的闭包问题
{% codeblock %}
for (var i = 1; i <= 5; i++) {
   setTimeout(function test() {
        console.log(i) //>> 6 6 6 6 6
    }, i * 1000);
}
{% endcodeblock %}
上述代码本意是希望：每隔一秒依次输出“1 2 3 4 5”，结果却变成输出“6 6 6 6 6 ”。由于setTimeout是异步操作，JS主线程会执行完同步任务（for循环）。此时根据作用域链上变量查找机制，setTimeout第一个参数的函数体内的i引用了全局作用域里面的i，当for循环完毕后，i的值为6，所以输出了“6 6 6 6 6 ”。

## 解决方法
- 闭包
{% codeblock %}
for (var i = 1; i <= 5; i++) {
  (function(j) {//包了一层IIFE形式的函数，这个函数是闭包
    setTimeout(function test() {//函数体内的j引用了外层匿名函数的参数j
      console.log(j); //>> 1 2 3 4 5
    }, j * 1000);
  })(i);
}
{% endcodeblock %}

- 作用域
{% codeblock %}
for (var i = 1; i <= 5; i++) {
   {
      let j = i;
      setTimeout(function test() {
           console.log(j) //>> 1 2 3 4 5
       }, j * 1000);
    }
}
{% endcodeblock %}
用let关键字包上一个作用域，也能和闭包一样解决问题达成目的

- 使用setTimeout第三个参数
{% codeblock %}
for ( var i=1; i<=5; i++) {
	setTimeout( function timer(j) {
		console.log( j );
	}, i*1000, i);
}
{% endcodeblock %}

{% note warning %}
setTimeout与setInterval的时间不准确，因为如果当前调用栈不为空，计时器事件对应的回调函数永远不会被执行（即使时间到了），所以用这两种方法做动画会不流畅、卡顿
{% endnote %}

# 参考资料
- 同步和异步，阻塞和非阻塞：https://coffe1891.gitbook.io/frontend-hard-mode-interview/1/1.2.7
- Event Loop：https://coffe1891.gitbook.io/frontend-hard-mode-interview/1/1.2.8
- setTimeout异步：https://www.cnblogs.com/ceceliahappycoding/p/10772351.html
- 面试时高频问到的“闭包”：https://coffe1891.gitbook.io/frontend-hard-mode-interview/1/1.2.5
- for循环内调用setTimeout：https://blog.csdn.net/u010200636/article/details/83061237