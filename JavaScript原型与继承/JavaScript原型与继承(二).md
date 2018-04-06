## 1.组合继承模式

我们常说对于引用类型的数据，不能直接赋值修改，因为即使赋值给另外一个变量后，这个变量的值实际保存的是这个引用类型的指针，该指针依然指向的是这个引用类型所在堆中的值，换句话说，该指针指向引用类型原型。

我觉得这也是由于js原型链本身特性所造成的一种简单的继承。

前一篇文章我们有看到组合继承，就如创建对象的组合构造模式一样。

温习组合模式：

```
// 组合构造模式，即合并构造函数模式和原型模式

// 1. 这一步是构造函数模式
function Test(name){
    this.name = name
}

// 2. 这一步是原型模式
Test.prototype = {
    // 此处最好将原型指向构造函数本身，虽然影响不大，具体解释前一章节有说到
    constructor: Test,
    getName() {
        console.log(this.name)
    }
}
```

同理，组合继承也是类似（我们依旧假设继承与被继承的2个对象为Child和Parent），将Child的原型重写并指向给Parent实例的原型

```
// 组合继承，即借用构造函数和重写原型的方式
function Parent(name) {
    this.name = name
    this.colors = ['red', 'green']
}

Parent.prototype.getName = function() {
    console.log(this.name)
}

function Child(name) {
    // 1.借用了 Parent构造函数，将name和colors属性“引用到”Child构造函数中
    // 这么做的目的是 使每个Child实例都拥有自己的name 和 colors
    Parent.call(this, name)
}

// 2.将`Child`的原型重写并指向给`Parent`实例的原型
Child.prototype = new Parent()
```

这样便达到了Child的所有实例，都拥有Parent实例的属性和方法，且这些属性和方法不是共享的，是实例本身所拥有的。

但这样同样会带来一个问题，Parent的构造函数会运行2次

```
// 第一次，将`Child`的原型重写并指向给`Parent`实例的原型
Child.prototype = new Parent()

// 第二次，实例化Child的时候，会调用Child构造函数，
// 此时，会再次调用Parent构造函数，克隆一份name和colors到Child中
let ym = new Child('ym')
```

注意：这个问题不仅仅是Parent的构造函数会运行2次的问题，还有一个问题是，Parent构造函数中的name和colors会存在2份，因为每一次调用Parent构造函数都会创建Parent的实例。

第一份存在于ym实例中，我们可以打印

```
let ym = new Child('ym')
ym.name // ym
ym.colors = // ['red', 'green']
```

如图所示

![image](http://olphpb1zg.bkt.clouddn.com/qq1.png)

第二份存在于Child.prototype

![image](http://olphpb1zg.bkt.clouddn.com/qq2.png)

这里我们可以看到Child.prototype中的name为undefined，因为我们第一次初始化Parent时，并没有传参。

为了印证这一点，我们前一章说过，delete操作符可以使得查找属性继续顺着原型链往上，（delete是删除当前对象上的属性）

这时，我们在第一次初始化的时候传一个参数name 为 test

```
Child.prototype = new Parent('test')
```

然后再删除实例ym上的name，调用getName打印当前name值

```
delete ym.name
ym.getName()  // test 符合预期
```

这个问题造成的原因前面也说过了，就是因为初始化了2次Parent构造函数。那究竟哪一次是多余的呢？答案是第一次。

## 2.合理的继承模式

想想继承是为了什么？就是为了拥有父类的所有属性和方法而又不造成原型链的“污染和浪费”，同时父类原型和子类原型又可以很好的扩展，那我们何不简单粗暴的把父类原型克隆一份，不需要使用prototype指来指去？，6月8日更新：克隆一份的说法是错误的理解，继承的核心是原型链，父类扩展后，子类也相应得到扩展，而克隆做不到。

也就是在第一次初始化的时候不使用new Parent()，而是直接克隆一份Parent.prototype赋值给Child.prototype

那么这里会有疑问，Parent的实例上的属性不就没有被Child继承了吗？答案是依旧被继承了，在第二次初始化的时候。
依旧是上面的例子改造，完整的示例：

```
function Parent(name) {
    this.name = name
    this.colors = ['red', 'green']
}

Parent.prototype.getName = function() {
    console.log(this.name)
}

function Child(name) {
    Parent.call(this, name)
}

Child.prototype = Object.create(Parent.prototype)

// 这里我们依旧手动指定构造函数，为了便于区分实例与构造函数的关系
Child.prototype.constructor = Child

let ym = new Child('ym')
console.log(ym)
```

> Object.create() 方法使用指定的原型对象和其属性创建了一个新的对象。

查看下结果：

![image](http://olphpb1zg.bkt.clouddn.com/qq3.png)

发现这个方法也是有瑕疵的，查找的时候多了一层object，原因是Object.create本身会返回一个新的对象实例，该实例的指针指向克隆的Parent.prototype

## 3.总结

继承的更合理的方式，是基于组合模式，去掉组合模式中多余的成分，即：

去掉第一次初始化Parent构造函数，使Parent的实例属性只存在于Child实例中

由于现在一般用ES6的特性写js，其中extends继承是经常用的，但是原理还有待深究(期待下一篇吧)......或许，这才是最合理的继承方法。