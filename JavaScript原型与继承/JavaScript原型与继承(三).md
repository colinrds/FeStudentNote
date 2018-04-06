## 前言

写这篇笔记的初衷，是想进一步了解ES6的特性extends是如何实现了继承，查看源码后发现核心的一段不能理解

```
// 赋值原型 
subClass.prototype = Object.create(superClass && superClass.prototype)

// 这一步是作甚？根据组合继承的逻辑，完全没有必要这一步？
if (superClass) 
    Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
```

由于不太清楚`__proto__`与原型prototype的含义，以及Object，Function的关系，便做进一步深究。

题外话：我并不觉得深究一件事有何不值得，就像是“走火入魔”般，何况这件事是很多人都曾做过的。我相信我会喜欢上深究一个问题这样一个过程而不是结果。

## 概述

这里我想要理清楚的问题是：

1. Object，Function，prototype，`__proto__`究竟是怎样的关系，怎么得来？
2. ES5合理继承的方式（即前篇文章所提到的组合继承）与ES6extends有何异同？ES6源码剖析

## 一、Object，Function，prototype，`__proto__`之间的关系

### 1.理解对象

我们常说，“js是面向对象的，因为它也可以拥有自己的属性和方法...”，其实说了跟没说没啥区别，反正我还是没理解。

我觉得最能解释它的是：Object.prototype是一切对象和函数的根源，一张图来证明我的观点：

![image](http://olphpb1zg.bkt.clouddn.com/www1.png)

为什么这么说？我们会发现，每当我们在控制台打印一个对象的时候，都能顺着原型链找到图中所有的内容。并且：

```
// 说明Object.prototype不是任何一个构造函数的实例
Object.prototype.__proto__ === null
```

### 2.谁构造了谁

js原生内置了部分构造函数，其中就包含了Object和Function

> Object、Function、Boolean、Number、String、Array、Date、RegExp、Error

我们知道，当我们想要一个子类拥有父类的属性和方法时，会先创建一个父类Parent，其实这个构造函数Parent形式等价于内置的构造函数

那Parent和这些内置构造函数继承的谁呢？答案是Function本身，换句话说所有的构造函数都是Function的实例，如图所示：

![image](http://olphpb1zg.bkt.clouddn.com/www2.png)

依照我们前面的说法，实例的`__proto__`始终指向构造函数的prototype没错。

那Function又是谁构造而来？我发现我真是蠢到了极点，图中不是答案么，Function是自己的实例，也就是Function由自己构造的，听起来像是科幻小说。

既然这样，那构造Function的时候，它的原型从哪继承而来？这里我不得不引用别人的原文：

> Function.prototype是Object的实例对象

虽然我们从控制台印证了这个观点

![image](http://olphpb1zg.bkt.clouddn.com/www3.png)

但是这里我还是得先弄清楚prototype和`__proto__`。最终，在别人的文章中找到了似乎可以印证的观点

> prototype是函数的一个属性（每个函数都有一个prototype属性），指向一个对象
我们先接受这个观点，那么这个对象指向的谁呢？创建函数的方式有3种

- 通过Function构造函数
- 字面量创建
- 直接声明

![image](http://olphpb1zg.bkt.clouddn.com/www4.png)

从图中我们可以看到，所有function实例的prototype都是指向Object.prototype

```
// 这里的a其实形式上等价于Function
a.prototype.__proto__ === Function.prototype.__proto__ === Object.prototype
```

> `__proto__`是一个对象拥有的内置属性，指向于它所对应的原型对象，原型链正是基于`__proto__`才得以形成(note：不是基于函数对象的属性prototype)

### 3.小总结

我们知道了`__proto__`和prototype的区别，那我们自然而然的得出：

```
// 所有构造函数都是Function的实例
Object.__proto__ === Function.prototype
Function.__proto__  === Function.prototype

// 所有Function都有一个prototype指向Object.prototype
// 也就是说，Function.prototype都是Object.prototype的实例
// 这一点我不是很确定，但从控制台看到确实如此
Function.prototype.__proto__ === Object.prototype

// Object.prototype到达了根源，不指向任何谁，原型链到此就结束了
Object.prototype.__proto__ === null
```

因此，关于谁构造了谁这个问题，答案是：

> 所以，是先有的Object.prototype,再有的Function.prototype，再有的Function和Object函数对象

最后再带上一张图来加深理解，此图来源Javascript中Function,Object,Prototypes,proto等概念详解

![image](http://olphpb1zg.bkt.clouddn.com/www5.png)


### 4.疑问

当我们通过3种方式创建函数的时候，它们的构造函数是一样的吗？

![image](http://olphpb1zg.bkt.clouddn.com/www6.png)

由上图可以发现，通过3种方式创建函数，它们实质是一样的，都是调用Function构造函数来创建，只不过创建的时候匿名不匿名的问题

```
a.constructor === b.constructor === c.constructor === Function
```

包括通过字面量创建的对象和通过构造函数创建对象他们的区别，只不过是多了一层中间构造函数而已

```
// 1.通过构造函数创建对象
function Extra(name) {
    this.name = name
}

var instance = new Extra('hehe')

// 2.通过字面量创建
var instance2 = {
    name: 'hehe2'
}

// instance与instance2的区别
instance.__proto__ === Extra.prototype
Extra.prototype.__proto__ === Object.prototype

// 而
instance2.__proto__ === Object.prototype

// 可以发现instance就多了一层构造函数Extra的原型，我们还可以知道，原型链就是基于__proto__才得以依次往上查找
```

> 区别我想肯定不止这些，因为我还不知道创建函数是怎样的一个过程，包括new的时候，具体发生了哪些细节，大致我只知道，new一个构造函数的时候，在其内部完成了原型链的连接（即继承至Object.prototype，可能是通过this = {}实现的），并且赋值了this的指向。而通过字面量创建的时候是没有这些过程的。

## 二、ES5合理继承的方式（即组合继承）与ES6 extends的异同

阮一峰老师的教程里有说到

> ES5 的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。
> ES6 的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。
也就是说ES6中子类是没有this的，必须通过super得到父类的实例对象this，这是结果，那么实现呢？

```
// 简单的extends继承
 class Parent{
   // static属性和方法 
   static sex = 'man'
   static getSex() {
     console.log(Parent.sex)
   }

   // 构造函数 
   constructor(name) {
     this.name = name
   }

   // 非静态方法
   say() {
     console.log(this.name)
   }
 }

 class Child extends Parent {
   static sex = 'women'

   constructor(name, age) {
      super(name)

      this.age = age
   }

   say() {
     console.log(this.name, this.age)
   }
 }
```

编译后查看源码

![image](http://olphpb1zg.bkt.clouddn.com/www7.png)

会发现es6 class 继承实现通过这4个方法：`_createClass`，`_possibleConstructorReturn`， `_inherits`, _classCallCheck，那么extends是如何继承，包括super是如何获取this的，我们来一步步解析。

### 1.编译后的Parent

```
var Parent = function () {

  // 函数内部声明一个同名的构造函数
  function Parent(name) {

    // 检查当前的this是否是构造函数的实例，也就是说必须通过new的方式调用构造函数
    // 而不能是像调用函数一样直接调用，因为这样是不会生成实例的this
    _classCallCheck(this, Parent);

    // 将构造函数的属性赋值给当前实例的this
    this.name = name;
  }

  // 将Parent中的静态方法直接赋值Parent构造函数
  _createClass(Parent, null, [{
    key: 'getSex',
    value: function getSex() {
      console.log(Parent.sex);
    }
  }]);

  // 将Parent中的非静态方法赋值Parent的原型，也就是Parent.prototype
  _createClass(Parent, [{
    key: 'say',
    value: function say() {
      console.log(this.name);
    }
  }]);

  return Parent;
}();
```

Parent做了2件事情，也可以说所有的通过Class关键字声明的函数做了2件事

- 在函数里面声明创建了一个同名的构造函数，将constructor以外的static声明的方法和非static声明的方法分别挂载到构造函数本身和构造函数的原型上
- 该函数会返回这个同名的构造函数，这里是采用寄生模式创建的构造函数。这里还做了校验，必须通过new来调用，否则抛出异常

### 2.constructor以外的方法是如何挂载的

```
// 用该方法实现
var _createClass = function () { 

    // 重写了es5的defineProperties方法，至于为什么会重写，后面解释
    // 该方法就是遍历props，然后利用defineProperty，依次给target定义属性
    function defineProperties(target, props) { 
        for (var i = 0; i < props.length; i++) { 
            var descriptor = props[i]; 

            // 是否可枚举，默认为false ？也就是说把所有的方法都置为不可枚举
            descriptor.enumerable = descriptor.enumerable || false; 
            descriptor.configurable = true; 

            if ("value" in descriptor) 
                descriptor.writable = true; 

            Object.defineProperty(target, descriptor.key, descriptor); 
        } 
    } 

    // 返回一个函数，如果是static方法就定义在Constructor上，否则就定义在Constructor.prototype上
    // 我们可以对应到Parent里调用_createClass 时，对于静态和非静态方法的传参
    return function (Constructor, protoProps, staticProps) { 
        if (protoProps) 
            defineProperties(Constructor.prototype, protoProps); 

        if (staticProps) 
            defineProperties(Constructor, staticProps); 

        return Constructor; 
    }; 
}();
```

我们可以看到，ES6把所有定义在构造函数或原型中的方法都定义为不可枚举，而属性是通过默认赋值可枚举的。为什么？谁能解释下。。。。

### 3.插播一条小广告，弄清楚数据属性

数据属性有4个（这里只是简要的带过）

- value 属性的值
- enumerable 是否可枚举
- configurable 是否可修改
- writable 是否可写

我们在定义对象或者赋值对象属性的时候，通常是不知道这些数据属性的，因为默认情况下，都是true

![image](http://olphpb1zg.bkt.clouddn.com/www8.png)

但是在有些时候，我们是不希望属性是可枚举的，就像前一章说过的组合继承，手动赋值constructor

```
// 此时constructor也是可枚举的
Child.prototype.constructor = Child
```

另外，通过Object.defineProperty定义属性时，如果不指定数据属性，默认情况下都为false

![image](http://olphpb1zg.bkt.clouddn.com/www9.png)

根据这些解释，就可以理解前文中_createClass为什么要重写defineProperties方法了。

### 4.编译后的Child

同理，Child与Parent一样，都会有相同的2个步骤（前面说的2件事情），不同的是

```
var Child = function (_Parent) {
  function Child(name, age) {
    // ...
    // 2.这里我想就是阮一峰老师文章里有说的
    // 拿到Parent实例的this，封装成Child子类的this
    var _this = _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name));
    // ...
  }

  // 1.这一步是关键的继承
  _inherits(Child, _Parent);
  // ...
}(Parent);
```

我们看到Child多做了2件事情，先说第1件事：

`_inherits`继承

`_inherits`也做了2件事

1. 就是我们组合继承中提到的，利用Object.create将子类的原型指向父类的原型
2. 将子类的`__proto__`指向父类构造函数，也就是说，认为子类是父类构造而来。

这里有点绕，其实就是这么个意思：你还记得组合继承中，在Child的构造函数里call了Parent一下么，将Parent上的属性都复制一份到Child的this中。那么在这里，ES6并不知道要call谁啊，所以只好将父类的构造函数指给子类的`__proto__`，这样后面就只需要`Child.__proto__.call(this)`了。

```
function _inherits(subClass, superClass) { 
    // 校验代码...

    // 利用 Object.create创建实例对象，并将实例赋值给subClass.prototype
    // 并手动赋值constructor
    subClass.prototype = Object.create(superClass && superClass.prototype, { 
        constructor: { 
            value: subClass, 
            enumerable: false, 
            writable: true, 
            configurable: true 
        } 
    }); 

    // 这一步就是上面说的第2件事，将superClass构造函数赋值给subClass.__proto__，方便后面调用
    // 因为我们知道，所有的函数都是由Function构造而来
    // 也就是说如果这里不赋值，subClass.__proto__ === Function.prototype
    if (superClass) 
        Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; 
}
```

Child做的第2件事情

`_possibleConstructorReturn`调用父类构造函数

其实这里的表述有误，调用父类构造函数并不是`_possibleConstructorReturn`做的，`_possibleConstructorReturn`只是做了一个简单的校验。

```
// 调用父类构造函数
(Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name)
```

看到这一句就证明我前面的理解是对的，这一句在es5组合继承中就是

```
Parent.call(this, name)
```

只不过它不知道Parent是谁，所以就先把Parent赋值给了Child.__proto__。由于ES6中都是通过寄生模式来创建构造函数，这里call之后，返回的是Parent的实例，与组合继承的call并不一致。

### 5.总结

我们再回过头来看ES5与ES6继承的区别，其实ES5与ES6的继承没有太大的区别，其原理都是采用了组合继承，核心唯一不同的就是这个this值的问题，另外就是对定义静态方法做了封装(staitc)。

es5的继承，是我们手动写父类，子类手动call父类。

但是es6中的继承是抽象出来的语法糖，并不知道你这里哪个是父类哪个是子类，所以它得通过一个巧妙的方法来知道这个是父类这个是子类。Child.__proto__ === Parent或者调用Object.setPrototypeOf()。

之所以this不一样，是因为它们创建构造函数的机制不一样

- es5是直接声明，那样this就直接被初始化了，所以只能在子类里通过call来“丰富”它的this
- es6则是通过寄生模式（非工厂模式），返回的一个新的构造函数，当你call的时候，相当于new了这个新的构造函数，此时父类的构造函数看起来就是个闭包，因为他还要返回new后的实例对象。所以在子类中是直接就拿到了父类的实例对象，那么就将this指向了他，再赋值自己的属性。