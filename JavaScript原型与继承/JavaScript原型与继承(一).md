我曾尝试理解关于prototype的相关概念，最初理解起来晦涩难懂，加上当时用的地方又少。后面渐渐明白，当你需要了解一个东西的时候，刻意的去理解是没有本质的作用的，但是能在你的脑海里留下一丝印象，当你真正遇到的时候，会想起曾经看到过，时机成熟的时候再去理解，会有不少收获，轮番看个几遍，拿上实例解析，会发现豁然开朗。

本文阐述的相关内容：

- 创建对象的几种模式以及创建的过程
- 原型链prototype的理解，以及prototype与 `__proto__（[[Prototype]]）`的关系
- 继承的几种实现

## 1.常见模式与原型链的理解

### a.构造函数创建

```
function Test() {
    // 
}
```
流程

- 创建函数的时候会默认为Test创建一个prototype属性，Test.prototype包含一个指针指向的是Object.prototype
- prototype默认会有一个constructor，且Test.prototype.constructor = Test
- prototype里的其它方法都是从Object继承而来

![image](http://olphpb1zg.bkt.clouddn.com/r111.png)

```
// 调用构造函数创建实例
var instance = new Test()
```
此处的instance包含了一个指针指向构造函数的原型，（此处的指针在chrome里叫`__proto__`，也等于`[[Prototype]]`）

![image](http://olphpb1zg.bkt.clouddn.com/r122.png)

### b.原型模式

由上我们可以知道，默认创建的prototype属性只拥有constructor和继承至Object的属性，原型模式就是为prototype添加属性和方法

```
Test.prototype.getName = ()=> {
    alert('name')
}
```

此时的instance实例就拥有了getName方法，因为实例的指针是指向Test.prototype的

```
instance.__proto__ === Test.prototype
```

如下图所示

![image](http://olphpb1zg.bkt.clouddn.com/r133.png)

这里我们可得知：实例instance与构造函数之间是通过原型prototype来相关联的。

### c.组合模式

这种模式我们用的最多，其实也是原型模式的另一种写法，只不过有一点小区别而已

```
function Test() {}

Test.prototype = {
    getName() {
        alert('name')
    }
}
```
我们经常会这么直接重写prototype方法，由上我们可知，prototype会默认自带constructor属性指向构造函数本身，那么重写以后呢？

```
Test.prototype.constructor === Object 
// 而并不等于Test了
// 因为重写以后相当于利用字面量方式创建一个实例对象，这个实例的构造函数是指向Object本身的
```

当然我们也可以手动赋值constructor

```
Test.prototype = {
    constructor: Test,
    getName() {
        alert('name')
    }
}
```

那么又会有疑问了constructor要不要有何意义？我觉得constructor意义仅仅是为了来鉴别原型所属的构造函数吧。

当需要获取某个属性的时候，会先从实例中查找，没有就根据指针所指向的原型去查找，依次向上，直到实例的指针`__proto__`指向为null时停止查找，例如：

```
// 1 读取name
instance.name 

// 2 instance.__proto__ === Test.prototype
Test.prototype.name

// 3 Test.prototype.__proto__ === Object.prototype
Object.prototype.name

// 4
Object.prototype.__proto__ === null
```

当找到了这个属性就会直接返回，而不会继续查找，即使这个属性值为null，想要继续查找，我们可以通过delete操作符来实现。

由这里我们自然可以想到Array, Date, Function, String，都是一个构造函数，他们的原型的指针都是指向Object.prototype，它们就像我这里定义的Test一样，只不过是原生自带而已

### d.几个有用的方法

Object.getPrototypeOf() 获取某个实例的指针所指向的原型

```
Object.getPrototypeOf(instance) === Test.prototype
```

hasOwnProperty 判断一个属性是存在于实例中还是存在于原型中，如图所示：

![image](http://olphpb1zg.bkt.clouddn.com/r144.png)

in操作符，无论该属性是否可枚举

```
'name' in instance  // true
'getName' in instance // true
```

无论属性是在实例中，还是在原型中都返回true，所以当我们需要判断一个属性存在与实例中，还是原型中有2种办法

```
// 一种就是使用hasOwnProperty判断在实例中
// 另一种判断在原型中
instance.hasOwnProperty('getName') === false && 'getName' in instance === true
```

for ... in操作符也是一样的，但只会列出可枚举的属性，ie8版本的bug是无论该属性是否可枚举，都会列出

![image](http://olphpb1zg.bkt.clouddn.com/r155.png)

name是在实例中定义的，getName是在原型中定义的

Object.keys()则不一样，它返回一个对象上所有可枚举的属性，仅仅是该实例中的

```
Object.keys(instance)
// ["name"]
```

### e.总结

以上讨论了构造函数，原型和实例的关系：

每个构造函数都有原型对象
每个原型对象都有一个constructor指针指向构造函数
每个实例都有一个`__proto__`指针指向原型

## 2.继承

继承的实质是利用构造函数的原型 = 某个构造函数的实例，以此来形成原型链。例如

```
// 定义父类
function Parent() {}
Parent.prototype.getName = ()=> {
    console.log('parent')
}
// 实例化父类
let parent = new Parent()

// 定义子类
function Child() {}
Child.prototype = parent 
// 实例化子类
let child = new Child()

child.getName() // parent
// 此时
child.constructor === parent.constructor === Parent
```

### a.最经典的继承模式

```
function Parent(name) {
    this.name = name
    this.colors = ['red']
}
Parent.prototype.getName = function() {
    console.log(this.name)
}
// 实例化父类
let parent = new Parent()

function Child(age, name) {
    Parent.call(this, name)
    this.age = age
}
Child.prototype = parent 
// 实例化子类
let child = new Child(1, 'aaa')
child.getName() // parent
```

这里会让我想到ES6中的class继承

```
class Parent {
    constructor(name) {
        this.name = name
        this.colors = ['red']
    }
    getName() {
        console.log(this.name)
    }
}

class Child extends Parent {
    constructor(age, name) {
        super(name)
    }
}

let child = new Child(1, 'aaa')
child.getName() // parent
```

其实是一个道理，这里我们不难想到，将Child.prototype指向parent实例，就是利用原型实现的继承，而为了每个实例都拥有各自的colors和name，也就是基础属性，在Child的构造函数中call调用了Parent的构造函数，相当于每次实例化的时候都初始化一遍colors和name，而不是所有实例共享原型链中的colors和name