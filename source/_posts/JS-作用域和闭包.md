---
title: JS基础 - 作用域和闭包
date: 2021-04-25 11:12:46
tags: JS
---

## 作用域
##### 1. 作用域是什么？
作用域是一套规则，这套规则用来管理引擎如何在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。

##### 2. 作用域主要有两种作用域模型：
第一种是最为普遍的，被大多数编程语言所采用的词法作用域；另外一种叫作动态作用域（比如 Bash 脚本、Perl 中的一些模式等在使用）
JavaScript采用的作用域模型是**词法作用域模型**

## 词法作用域
##### 1. 词法作用域
词法作用域就是定义在词法阶段的作用域。
无论函数在哪里被调用，也无论它如何被调用，词法作用域只由函数被声明时所处的位置决定

##### 2. 作用域查找
 作用域查找始终从运行时所处的最内部作用域开始，逐级向外或者说向上进行，直到遇见第一个匹配的标识符为止。

##### 3. 欺骗词法： 修改词法作用域的方法
- eval() 
会将传入的字符串当做代码来执行。在程序中动态生成了代码，可以在运行期修改书写期的词法作用域
- with()
将一个对象的引用当作作用域来处理。with 可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对 象的属性也会被处理为定义在这个作用域中的词法标识符。

## 函数作用域
##### 1. 函数中的作用域
属于这个函数的全部变量都可以在整个函数范围内使用及复用。
函数作用域可以隐藏作用域中的变量和函数，避免同名标识符之间的冲突

##### 2. 隐藏内部实现
规避冲突：
###### （1）全局命名空间
第三方库通常会在全局作用域中声明一个名字足够独特的变量，通常是一个对象。这个对象 被用作库的命名空间
###### （2）模块管理
通过依赖管理器 的机制将库的标识符显式地导入到另外一个特定的作用域中。这些工具利用作用域的规则强制所有标识符都不能注入到共享作用域中，而是保持在私有、无冲突的作用域 中，这样可以有效规避掉所有的意外冲突

##### 3. 函数表达式:
具名函数表达式：**(function foo(){ .. })**，foo只能在 ... 所代表的位置中 被访问，外部作用域则不行

匿名函数表达式：**(function(){ .. })**

立即执行函数表达式(IIFE):  **(function foo(){ .. })()**。改进后：**(function(){ .. }())**

## 块作用域
JS中的 if 、for语句不能创建块作用域，在其他语言中支持块作用域
##### 1. with
用 with 从对象中创建出的作用域仅在 with 声明中有效。

##### 2. try/catch
try/catch 的 catch 分句会创建一个块作用域，其中声明的变量仅在 catch 内部有效。

##### 3. let
let关键字可以将变量绑定到所在的任意作用域中，通常是 {...} 内部。
用 let 将变量附加在一个已经存在的块作用域上的行为是隐式的。
if (..) { let a = 2; } 会声明一个劫持了 if 的 { .. } 块的变量，并且将变量添加到这个块 中。
let 进行的声明不会在块作用域中进行提升。声明的代码被运行之前，声明并不“存在”

##### 4. const
const也可以创建块作用域变量，但是值是固定的，不能修改

## 变量提升

变量和函数的声明会从他们在代码中出现的位置移动到所在作用域的最上面。

```
if(!('a' in window)) {
    var a = 1;
}
console.log(a) // undefined
上面的代码等价于
var a;
if(!('a' in window)) {
    var a = 1;
}
console.log(a)
```

'a' in window  一直为true，所以不会进行 a = 1 的操作

函数和变量都会提升，但是函数会首先被提升，然后才是变量。遇到同名的，函数优先级更高
```
foo(); // 1

var foo;

function foo() {
  console.log(1);
}

foo = function() {
  console.log(2);
}
```
会输出1，不是2

## 闭包
**1.当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包。**

```
function foo() { 
    var a = 2;
    function bar() { 
        console.log( a ); // 2
    }
    bar(); 
}
foo();
```

上面的不是闭包，bar() 对 a 的引用的方法，只是词法作用域的查找规则，并没有在作用域外部执行

闭包的例子：

```
function foo() { 
    var a = 2;
    function bar() { 
        console.log( a );
    }
    return bar; 
}
var baz = foo();
baz(); // 2
```

在执行 foo() 的时候返回了 bar，调用 baz() 相当于是调用了 bar() ，bar() 涵盖了 foo() 作用域，并且在 foo 外部调用的时候，也可以获取到 a 的值。bar() 一直保持着对foo函数作用域的引用，这个引用就叫闭包

```
var fn;
function foo() {
    var a = 2;
    function baz() { 
        console.log( a );
    }
    fn = baz; // 将 baz 分配给全局变量 
}
function bar() {
    fn(); // 妈妈快看呀，这就是闭包!
}
foo();
bar(); // 2
```

这也是闭包。无论通过什么方式将内部函数传递到所在作用域之外，他都会保持对原作用域的引用，无论在何处执行这个函数都会使用闭包。


**2. 在定时器、事件监听器、 Ajax 请求、跨窗口通信、Web Workers 或者任何其他的异步(或者同步)任务中，只要使 用了回调函数，实际上就是在使用闭包。**

```
function wait(message) {
    setTimeout(function timer() { 
        console.log( message );
    }, 1000 );
}
wait( "Hello, closure!" );
```

这是闭包，wait(..) 执行 1000 毫秒后，它的内部作用域并不会消失，timer 函数依然保有 wait(..)作用域的闭包。

```
function setupBot(name, selector) {
    $( selector ).click( function activator() {
        console.log( "Activating: " + name ); 
    } );
}
setupBot( "Closure Bot 1", "#bot_1" );
```

这也是闭包。

##### 3. 循环和闭包

```
for (var i=1; i<=5; i++) { 
    setTimeout( function timer() {
        console.log( i ); 
    }, i*1000 );
}
// 输出 5 个 6
```

只有一个全局的 i，和不适用循环，直接调用了 5 次 timer() 是一样的。
要实现依次打印 1 ~ 5 ，要通过使用闭包这样改进：

```
for (var i=1; i<=5; i++) { 
    (function() {
        var j = i;
        setTimeout( function timer() {
            console.log( j ); 
        }, j*1000 );
    })(); 
}
```

let 可以用来劫持块级作用域，改成下面的形式也可以实现功能

```
for (let i=1; i<=5; i++) { 
    setTimeout( function timer() {
        console.log( i ); 
    }, i*1000 );
}
```

##### 4. 模块
闭包可以用来实现模块模式。用闭包模拟私有方法

```
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];
    function doSomething() { 
        console.log( something );
    }
    function doAnother() {
        console.log( another.join( " ! " ) );
    }
    return {
        doSomething: doSomething, doAnother: doAnother
    };
}
var foo = CoolModule(); 
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

CoolModule()是一个函数，通过调用他来产生模块实例。
CoolModule() 返回一个用对象字面量语法 { key: value, ... } 来表示的对象。这 个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量是隐 藏且私有的状态。可以将这个对象类型的返回值看作本质上是模块的公共 API

模块模式必须具备两个条件：
- 必须有外部的封装函数，该函数至少必须被调用一次（每次调用都会创建新的模块实例）
- 封装函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或修改私有的状态

模块的两个特征：
- 为创建内部作用域而调用了一个包装函数;
- 包装函数的返回值必须至少包括一个对内部函数的引用，这样就会创建涵盖整个包装函数内部作用域的闭 包。

##### 5. 现代模块机制
大多数模块依赖加载器 / 管理器本质上都是将这种模块定义封装进一个友好的 API。

##### 6. ES中的模块
ES6 中为模块增加了一级语法支持。通过模块系统进行加载时，ES6 会将文件当作独立的模块来处理（一个文件一个模块）。每个模块都可以导入其他模块或特定的 API 成员，同样也可以导出自己的 API 成员。
bar.js

```
function hello(who) {
    return "Let me introduce: " + who;
}
export hello;
```

 foo.js

```
// 仅从 "bar" 模块导入 hello() 
import hello from "bar";
var hungry = "hippo";
function awesome() { 
    console.log(hello( hungry ).toUpperCase() );
}
export awesome;
```

baz.js

```
// 导入完整的 "foo" 和 "bar" 模块
module foo from "foo"; 
module bar from "bar";
console.log(bar.hello( "rhino" )); // Let me introduce: rhino 
foo.awesome(); // LET ME INTRODUCE: HIPPO
```

import 可以将一个或多个 API 导入到当前作用域中，并分别绑定到一个变量上。
module 会将所有的 API 导入，并绑定到一个变量上。
export 会将当前模块的标识符导出为公共 API 。
模块文件中的内容会被当作好像包含在作用域闭包中一样来处理

## 动态作用域
JavaScript 中的作用域就是词法作用域，词法作用域是一套如何寻找变量，并在何处会找到变量的规则。词法作用域最重要的特征是它的定义过程发生在代码的书写阶段。

动态作用域并不关心函数和作用域是如何声明以及在何处声明的，只关心它们从何处调 用。

**JavaScript 并不具有动态作用域。它只有词法作用域**

##### 动态作用域和词法作用域的主要区别
词法作用域是在写代码或者说定义时确定的，而动态作用域是在运行时确定的。(this 也是!)词法作用域关注函数在何处声明，而动态作用域关注函数从何处调用。

## this词法
##### 1. this绑定
```
var obj = {
    id: "awesome",
    cool: function coolFn() { 
        console.log( this.id );
    } 
};
var id = "not awesome"
obj.cool(); // awesome
setTimeout( obj.cool, 100 ); // not awesome
```
setTimeout中调用时，cool() 函数丢失了同 this 之间的绑定。
解决办法
```
var obj = { 
    count: 0,
    cool: function coolFn() {
        var self = this;
        if (self.count < 1) {
            setTimeout( function timer(){
                self.count++;
                console.log( "awesome?" ); 
            }, 100 );
        } 
    }
};
obj.cool(); // awesome？
```
使用了词法作用域的规则，self只是一个可以通过 词法作用域和闭包进行引用的标识符，不关心 this 绑定的过程中发生了什么


##### 2. 箭头函数
箭头函数不会创建自己的this,它只会从自己的作用域链的上一层继承this
使用ES6的箭头函数
```
var obj = { 
    count: 0,
    cool: function coolFn() { 
        if (this.count < 1) {
            setTimeout( () => { // 
                console.log( "awesome?" ); 
            }, 100 );
        } 
    }
};
obj.cool(); // awesome?
```
箭头函数在涉及 this 绑定时的行为和普通函数的行为完全不一致。它放弃了所有普通 this 绑定的规则，取而代之的是用当前的词法作用域覆盖了 this 本来的值。当前作用域是 coolFn ，coolFn的this是obj

