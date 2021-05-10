---
title: JS-原型
date: 2021-05-10 22:37:01
tags:
---

每个对象都有一个 **\_\_proto__** 属性，是对于其他对象的引用（就是它的构造函数的 prototype 对象）。在获取一个对象的属性时，如果在当前对象中没找到，就会沿着原型链向上查找。原型链的终点是 **Object.prototype**

使用 **for..in** 遍历对象时原理和查找原型链类似，任何可以通过原型链访问到 (并且是 enumerable)的属性都会被枚举

## 属性设置和屏蔽

```
myObject.foo = "bar";
```

如果属性名 foo 既出现在 myObject 中也出现在 myObject 的原型链上层，那 么就会发生屏蔽。myObject 中包含的 foo 属性会屏蔽原型链上层的所有 foo 属性，因为 myObject.foo 总是会选择原型链中最底层的 foo 属性。

如果 foo 不直接存在于 myObject 中而是存在于原型链上层时， **myObject.foo = "bar"** 会出现三种情况：  
（1）如果在原型链上层存在名为foo的普通数据访问属性并且没有被标记为只读(writable:true)，那就会直接在 myObject 中添加一个名为 foo 的新属性，它是屏蔽属性。  
（2）如果在原型链上层存在foo，但是它被标记为只读(writable:false)，那么 无法修改已有属性或者在 myObject 上创建屏蔽属性。  如果运行在严格模式下，代码会 抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。(只读属性会阻止原型链下层 隐式创建(屏蔽)同名属性)  
（3）如果在原型链上层存在foo并且它是一个setter，那就一定会 调用这个 setter。foo 不会被添加到myObject，也不会重新定义 foo 这个 setter。

**隐式产生屏蔽：**

```
var anotherObject = { 
    a:2
};
var myObject = Object.create( anotherObject );
anotherObject.a; // 2 
myObject.a; // 2
anotherObject.hasOwnProperty( "a" ); // true 
myObject.hasOwnProperty( "a" ); // false
myObject.a++; // 隐式屏蔽! 
anotherObject.a; // 2
myObject.a; // 3 
myObject.hasOwnProperty( "a" ); // true
```

## 构造函数

```
function Foo() { 
    // ...
}
var a = new Foo();
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

调用 new Foo() 时会创建 a，其中的一步就是给 a 一个内部的原型链接，关联到 Foo.prototype 指向的那个对象。

函数本身并不是构造函数，然而，当你在普通的函数调用前面加上 new 关键字之后，就会把这个函数调用变成一个“构造函数调用”。实际上，new 会劫持所有普通函数并用构造对象的形式来调用它。
在 JavaScript 中对于“构造函数”最准确的解释是，所有带 new 的函数调用。
函数不是构造函数，但是当且仅当使用 new 时，函数调用会变成“构造函数调用”。

```
function Foo(name) { 
    this.name = name;
}
Foo.prototype.myName = function() { 
    return this.name;
};
var a = new Foo( "a" );
var b = new Foo( "b" ); 
a.myName(); // "a"
b.myName(); // "b"
```

看起来 a.constructor === Foo 为真意味着 a 确实有一个指向 Foo 的 .constructor 属性，但其实不是。  
实际上，.constructor 被委托给了 Foo.prototype，而 Foo.prototype.constructor 默认指向 Foo  

```
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
var a1 = new Foo();
a1.constructor === Foo; // false! 
a1.constructor === Object; // true!
```

a1 没有 constructor 属性，会委托给原型链上的 Foo.prototype，但是这个对象也没有.constructor 属性，会继续委托给原型链顶端的 Object.prototype，这个对象有 .constructor 属性，指向内置的 Object(..) 函数。

.constructor 并不是一个不可变属性。它是不可枚举的，但是它的值是可写的(可以被修改)
也可以手动给 Foo.prototype 添加一个 .constructor 属性，不过这需要手动添加一个符合正常行为的不可枚举属性，如：  

```
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
// 需要在 Foo.prototype 上“修复”丢失的 .constructor 属性 
// 新对象属性起到 Foo.prototype 的作用 
Object.defineProperty( Foo.prototype, "constructor" , {
    enumerable: false,
    writable: true,
    configurable: true,
    value: Foo // 让 .constructor 指向 Foo
} );
```


## 原型继承

```
function Foo(name) { 
    this.name = name;
}
Foo.prototype.myName = function() { 
    return this.name;
};
function Bar(name,label) { 
    Foo.call( this, name ); 
    this.label = label;
}
// 创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype 
Bar.prototype = Object.create( Foo.prototype );
// 注意!现在没有 Bar.prototype.constructor 了 
// 如果你需要这个属性的话可能需要手动修复一下它
Bar.prototype.myLabel = function() { 
    return this.label;
};
var a = new Bar( "a", "obj a");
a.myName(); // "a" 
a.myLabel(); // "obj a"
```

调用 Object.create(..) 会凭空创建一个“新”对象，并把新对象内部的原型对象(\_\_proto__)关联到你指定的对象，把元素的关联对象抛弃掉。

下面是错误的做法：  

```
// 和你想要的机制不一样! 
Bar.prototype = Foo.prototype;
// 基本上满足你的需求，但是可能会产生一些副作用
Bar.prototype = new Foo();
```

**Bar.prototype = Foo.prototype** 并不会创建一个关联到 Bar.prototype 的新对象，它只是让 Bar.prototype 直接引用 Foo.prototype 对 象。如果修改  Bar.prototype ，会影响到  Foo.prototype 。

**Bar.prototype = new Foo()** 会创建一个关联到 Bar.prototype 的新对象，但是如果Foo 有一些副作用（比如写日志、修改状态、注册到其他对象、给 this 添加数据属性，等等），就会影响到 Bar() 的“后代”。

ES6 添加了辅助函数 Object.setPrototypeOf(..)，可以用标准并且可靠的方法来修改关联。

```
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.ptototype = Object.create( Foo.prototype );
// ES6 开始可以直接修改现有的 Bar.prototype 
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

## 判断关系
使用 instanceof   

```
a instanceof Foo;
```

在 a 的整条原型链中是否有指向 Foo.prototype 的对象

使用 isPrototypeOf
```
Foo.prototype.isPrototypeOf( a ); // true
```

## 对象关联
原型机制就是存在于对象中的一个内部链接，它会引用其他对象。
这个对象的作用是：如果在对象上没有找到需要的属性或者方法引用，引擎就会继续在原型对象(\_\_proto__)关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的原型对象，以此类推。这一系列对象的链接被称为“原型链”。

创建关联：**Object.create(..)** 。  
Object.create(null) 会创建一个拥有空(或者说null)__proto__链接的对象，这个对象无法进行委托