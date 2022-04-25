---
title: JavaScript闭包
date: 2022-04-25 14:55:07
excerpt: 由于JavaScript的闭包（closure）是其最强大的特性，jQuery、Vue.js库都使用了闭包特性来实现，本篇文章总结了一下博主自己对JavaScript闭包特性的学习和理解
index_img: https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/JavaScript_Closure/closure2.png
tags:
  - JavaScripy
  - 闭包
categories:
  - JavaScript教程
---

# 前言
由于JavaScript的闭包（closure）是其最强大的特性，jQuery、Vue.js库都使用了闭包特性来实现，本篇文章总结了一下博主自己对JavaScript闭包特性的学习和理解

# 闭包
## 闭包的例子
我们先简单展示一个闭包的代码，让大家先观察一下闭包代码的形式
{% codeblock %}
function func(){//func1引用了它外层的变量a，因此func成为了闭包
    let a="coffe";
    function func1(){
        console.log(a);//访问了外层函数func体内的变量a
    }
    func1();
}

func();
{% endcodeblock %}
上述代码中，func即为闭包

## 闭包的定义
- 不同作者对于闭包的定义都有不同的描述，理解其核心在于记住**产生闭包的时机**
{% note info %}
**内层的作用域访问它外层函数作用域里面的参数/变量/函数时，闭包就产生了**
{% endnote %}

- **闭包也是一种作用域**
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/JavaScript_Closure/closure1.png)
在chrome浏览器“开发者工具”的控制台可以看到闭包出现在`Scope`一栏，因此闭包也是一种作用域
{% note info %}
**闭包是一种作用域，它拷贝了一套外层函数作用域中被访问的参数、变量/函数，这个拷贝都是浅拷贝（引用）**
{% endnote %}

## 闭包的优点
- 访问其他函数内部的变量
{% codeblock %}
function outer() {	// outer为闭包
     var  a = '变量1'
     var  inner = function () {
            console.info(a)
     }
     return inner    // inner访问了outer函数作用域中的变量a
}
var  inner = outer()   
inner()   //"变量1"
{% endcodeblock %}
上述代码中，`inner`函数作用域访问了外层作用域函数`outer`中的变量`a`

- 闭包内部的变量无法被外部作用域访问和修改，可以实现软件设计上的**封装**
{% codeblock %}
//定义一个模块
function module(n) {
  //私有属性
  let name = n;
  //私有方法
  function getModuleName() {
    return name;
  }
  //私有方法
  function someMethod() {
    console.log("coffe1891");
  }
  //以一个对象的形式返回
  return {
    getModuleName: getModuleName,
    getXXX: someMethod
  };
}

let myapp = module("myModule");//定义一个模块
console.log(myapp.getModuleName()); //>> myModule
console.log(myapp.getXXX()); //>> coffe1891
{% endcodeblock %}
上述代码中，变量`name`、函数`getModuleName`与`someMethod`类似高级语言的私有属性和方法，无法被外部作用域访问和修改（除非提供返回的对象接口），只有`module`内部作用域可以访问，实现了设计上的“封装”

- 保护变量不被内存回收机制回收
{% codeblock %}
var report = function(src) {
    var img = new Image();
    img.src = src;
}
report('http://www.xxx.com/getClientInfo');//把客户端信息上报数据
{% endcodeblock %}
上述用于数据统计上报的代码，会丢失部分数据上报。原因是`Image`对象是`report`函数中的局部变量，当`report`函数调用结束后，`Image`对象随即被JS引擎垃圾回收器回收，而此时可能还没来得及发出http请求，导致上报数据请求失败
{% codeblock %}
var report = (function() {
    var imgs = [];//在内存里持久化
    return function(src) {
        var img = new Image();
        imgs.push(img);//引用局部变量imgs
        img.src = src;
    }
}());
report('http://www.xxx.com/getClientInfo');//把客户端信息上报数据
{% endcodeblock %}
使用闭包把`Image`对象封装起来，就可以解决数据丢失问题。此时，`imgs`变量被`report`函数作用域链所引用，不会在IIFE函数执行完成后，因为退出函数调用栈而被JS引擎垃圾回收器收回

## 闭包的缺点
- 过渡使用闭包会占用过多内存，甚至引起内存泄漏
{% codeblock %}
function A(){
    var count = 0;
    function B(){
       count ++;
       console.log(count);
    }
    return B;//函数B保持了对count的引用
}
var b = A();
b();//>> 1
b();//>> 2
b();//>> 3
{% endcodeblock %}
上述代码中，当执行完`var b = A();`之后，A函数的执行环境并没有被销毁，其中`count`变量被`b`的函数作用域链所引用，并没有因为函数A执行完毕退出函数调用栈而被JS引擎垃圾回收器回收，直至三次调用`b()`之后，**并且删除变量b或赋值为null**，`b`和`A`的执行环境才会被销毁
{% note warning %}
JavaScript中的垃圾回收规则：如果对象不再被引用，或者对象互相引用形成数据孤岛后且没有被孤岛之外的其他对象引用，那么这些对象将会被JS引擎的垃圾回收器回收；反之，这些对象一直会保存在内存中
{% endnote %}
由于闭包会引用包含它的外层函数作用域里的变量/函数，因此会比其他非闭包形式的函数占用更多内存。即使外层函数执行完毕退出函数调用栈时，由于外层函数作用域中的变量被引用着，并不会被JS引擎的垃圾回收器回收

## 避免闭包内存泄漏的方法
- 避免闭包导致内存泄漏的解决方法是，在函数A执行完毕退出函数调用栈之前，将不再使用的局部变量全部删除或者赋值为null
{% codeblock %}
这段代码会导致内存泄露
window.onload = function(){
    var el = document.getElementById("id");
    el.onclick = function(){
        alert(el.id);
    }
}
解决方法为
window.onload = function(){
    var el = document.getElementById("id");
    var id = el.id;                                      //解除循环引用
    el.onclick = function(){
        alert(id); 
    }
    el = null;                                          // 将闭包引用的外部函数中活动对象清除
}
{% endcodeblock %}

## 闭包的写法
- 循环中的闭包
{% codeblock %}
for (var i = 1; i <= 5; i++) {
  (function(j) {//包了一层IIFE形式的函数，这个函数是闭包
    setTimeout(function test() {//函数体内的j引用了外层匿名函数的参数j
      console.log(j); //>> 1 2 3 4 5
    }, j * 1000);
  })(i);
}
{% endcodeblock %}
- 模块化封装
{% codeblock %}
//定义一个模块
function module(n) {
  //私有属性
  let name = n;
  //私有方法
  function getModuleName() {
    return name;
  }
  //私有方法
  function someMethod() {
    console.log("coffe1891");
  }
  //以一个对象的形式返回
  return {
    getModuleName: getModuleName,
    getXXX: someMethod
  };
}

let myapp = module("myModule");//定义一个模块
console.log(myapp.getModuleName()); //>> myModule
console.log(myapp.getXXX()); //>> coffe1891
{% endcodeblock %}
- 返回新函数
{% codeblock %}
function sayHello2(name) {
    var text = "Hello " + name; // 局部变量

    var sayAlert = function() {
        console.log(text);
    };

    return sayAlert;
}

var say2 = sayHello2("coffe1891");
say2(); //>> Hello coffe1891
{% endcodeblock %}
调用`sayHello2()`函数返回了`sayAlert`，赋值给`say2`。`say2`是一个引用变量，指向一个函数本身，而不是指向一个变量

- 扩展全局对象
{% codeblock %}
function setupSomeGlobals() {
    //私有变量
    var num = 666;

    gAlertNumber = function() {//没有用var和let关键字声明，会成为全局对象的方法
        console.log(num);
    };

    gIncreaseNumber = function() {
        num++;
    };

    gSetNumber = function(x) {
        num = x;
    };
}

setupSomeGlobals();
gAlertNumber(); //>> 666

gIncreaseNumber();
gAlertNumber(); //>> 667

gSetNumber(1891);
gAlertNumber(); //>> 1891
{% endcodeblock %}
三个全局函数`gAlertNumber`，`gIncreaseNumber`，`gSetNumber`指向了同一个闭包，因为它们是在同一次`setupSomeGlobals()`调用中声明的。它们所指向的闭包是与`setupSomeGlobals()`函数关联一个作用域，该作用域包括了`num`变量的拷贝。也就是说，这三个函数操作的是同一个`num`变量

- 延长局部变量生命
{% codeblock %}
var report = (function() {
    var imgs = [];//在内存里持久化
    return function(src) {
        var img = new Image();
        imgs.push(img);//引用局部变量imgs
        img.src = src;
    }
}());
report('http://www.xxx.com/getClientInfo'); //把客户端信息上报数据
{% endcodeblock %}
闭包把`Image`对象封闭起来，就可以解决数据丢失的问题


# 参考资料
闭包造成的内存泄露如何解决：https://www.cnblogs.com/yanjianjiang/p/13881231.html
面试时高频问到的“闭包”：https://coffe1891.gitbook.io/frontend-hard-mode-interview/1/1.2.5
