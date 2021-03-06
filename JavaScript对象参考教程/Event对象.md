事件是一种异步编程的实现方式，本质上是程序各个组成部分之间的通信。DOM支持大量的事件，本节介绍DOM的事件编程。

## EventTarget接口

DOM的事件操作（监听和触发），都定义在`EventTarget`接口。`Element`节点、`document`节点和`window`对象，都部署了这个接口。此外，XMLHttpRequest、AudioNode、AudioContext等浏览器内置对象，也部署了这个接口。

该接口就是三个方法，`addEventListener`和`removeEventListener`用于绑定和移除监听函数，`dispatchEvent`用于触发事件。

### addEventListener()

`addEventListener`方法用于在当前节点或对象上，定义一个特定事件的监听函数。

```javascript
target.addEventListener(type, listener[, useCapture]);
```

上面是使用格式，addEventListener方法接受三个参数。

- type，事件名称，大小写不敏感。

- listener，监听函数。指定事件发生时，会调用该监听函数。

- useCapture，监听函数是否在捕获阶段（capture）触发（参见后文《事件的传播》部分）。该参数是一个布尔值，默认为false（表示监听函数只在冒泡阶段被触发）。老式浏览器规定该参数必写，较新版本的浏览器允许该参数可选。为了保持兼容，建议总是写上该参数。

下面是一个例子。

```javascript
function hello(){
  console.log('Hello world');
}

var button = document.getElementById("btn");
button.addEventListener('click', hello, false);
```

上面代码中，addEventListener方法为button节点，绑定click事件的监听函数hello，该函数只在冒泡阶段触发。

可以使用addEventListener方法，为当前对象的同一个事件，添加多个监听函数。这些函数按照添加顺序触发，即先添加先触发。如果为同一个事件多次添加同一个监听函数，该函数只会执行一次，多余的添加将自动被去除（不必使用removeEventListener方法手动去除）。

```javascript
function hello(){
  console.log('Hello world');
}

document.addEventListener('click', hello, false);
document.addEventListener('click', hello, false);
```

执行上面代码，点击文档只会输出一行“Hello world”。

如果希望向监听函数传递参数，可以用匿名函数包装一下监听函数。

```javascript
function print(x) {
  console.log(x);
}

var el = document.getElementById("div1");
el.addEventListener("click", function(){print('Hello')}, false);
```

上面代码通过匿名函数，向监听函数print传递了一个参数。

### removeEventListener()

removeEventListener方法用来移除addEventListener方法添加的事件监听函数。

```javascript
div.addEventListener('click', listener, false);
div.removeEventListener('click', listener, false);
```

removeEventListener方法的参数，与addEventListener方法完全一致。它对第一个参数“事件类型”，也是大小写不敏感。

注意，removeEventListener方法移除的监听函数，必须与对应的addEventListener方法的参数完全一致，而且在同一个元素节点，否则无效。

### dispatchEvent()

`dispatchEvent`方法在当前节点上触发指定事件，从而触发监听函数的执行。该方法返回一个布尔值，只要有一个监听函数调用了`Event.preventDefault()`，则返回值为`false`，否则为`true`。

```javascript
target.dispatchEvent(event)
```

`dispatchEvent`方法的参数是一个`Event`对象的实例。

```javascript
para.addEventListener('click', hello, false);
var event = new Event('click');
para.dispatchEvent(event);
```

上面代码在当前节点触发了`click`事件。

如果`dispatchEvent`方法的参数为空，或者不是一个有效的事件对象，将报错。

下面代码根据`dispatchEvent`方法的返回值，判断事件是否被取消了。

```javascript
var canceled = !cb.dispatchEvent(event);
  if (canceled) {
    console.log('事件取消');
  } else {
    console.log('事件未取消');
  }
}
```

## 监听函数

监听函数（listener）是事件发生时，程序所要执行的函数。它是事件驱动编程模式的主要编程方式。

DOM提供三种方法，可以用来为事件绑定监听函数。

### HTML标签的on-属性

HTML语言允许在元素标签的属性中，直接定义某些事件的监听代码。

```html
<body onload="doSomething()">
<div onclick="console.log('触发事件')">
```

上面代码为`body`节点的`load`事件、`div`节点的`click`事件，指定了监听函数。

使用这个方法指定的监听函数，只会在冒泡阶段触发。

注意，使用这种方法时，`on-`属性的值是将会执行的代码，而不是“监听函数”。

```html
<!-- 正确 -->
<body onload="doSomething()">

<!-- 错误 -->
<body onload="doSomething">
```

一旦指定的事件发生，`on-`属性的值是原样传入JavaScript引擎执行。因此如果要执行函数，不要忘记加上一对圆括号。

另外，Element节点的`setAttribute`方法，其实设置的也是这种效果。

```javascript
el.setAttribute('onclick', 'doSomething()');
```

### Element节点的事件属性

Element节点有事件属性，可以定义监听函数。

```javascript
window.onload = doSomething;

div.onclick = function(event){
  console.log('触发事件');
};
```

使用这个方法指定的监听函数，只会在冒泡阶段触发。

### addEventListener方法

通过`Element`节点、`document`节点、`window`对象的`addEventListener`方法，也可以定义事件的监听函数。

```javascript
window.addEventListener('load', doSomething, false);
```

addEventListener方法的详细介绍，参见本节EventTarget接口的部分。

在上面三种方法中，第一种“HTML标签的on-属性”，违反了HTML与JavaScript代码相分离的原则；第二种“Element节点的事件属性”的缺点是，同一个事件只能定义一个监听函数，也就是说，如果定义两次onclick属性，后一次定义会覆盖前一次。因此，这两种方法都不推荐使用，除非是为了程序的兼容问题，因为所有浏览器都支持这两种方法。

addEventListener是推荐的指定监听函数的方法。它有如下优点：

- 可以针对同一个事件，添加多个监听函数。

- 能够指定在哪个阶段（捕获阶段还是冒泡阶段）触发回监听函数。

- 除了DOM节点，还可以部署在window、XMLHttpRequest等对象上面，等于统一了整个JavaScript的监听函数接口。

### this对象的指向

实际编程中，监听函数内部的this对象，常常需要指向触发事件的那个Element节点。

addEventListener方法指定的监听函数，内部的this对象总是指向触发事件的那个节点。

```javascript
// HTML代码为
// <p id="para">Hello</p>

var id = 'doc';
var para = document.getElementById('para');

function hello(){
  console.log(this.id);
}

para.addEventListener('click', hello, false);
```

执行上面代码，点击p节点会输出para。这是因为监听函数被“拷贝”成了节点的一个属性，使用下面的写法，会看得更清楚。

```javascript
para.onclick = hello;
```

如果将监听函数部署在Element节点的on-属性上面，this不会指向触发事件的元素节点。

```html
<p id="para" onclick="hello()">Hello</p>
<!-- 或者使用JavaScript代码  -->
<script>
  pElement.setAttribute('onclick', 'hello()');
</script>
```

执行上面代码，点击p节点会输出doc。这是因为这里只是调用hello函数，而hello函数实际是在全局作用域执行，相当于下面的代码。

```javascript
para.onclick = function(){
  hello();
}
```

一种解决方法是，不引入函数作用域，直接在on-属性写入所要执行的代码。因为on-属性是在当前节点上执行的。

```html
<p id="para" onclick="console.log(id)">Hello</p>
<!-- 或者 -->
<p id="para" onclick="console.log(this.id)">Hello</p>
```

上面两行，最后输出的都是para。

总结一下，以下写法的this对象都指向Element节点。

```javascript
// JavaScript代码
element.onclick = print
element.addEventListener('click', print, false)
element.onclick = function () {console.log(this.id);}

// HTML代码
<element onclick="console.log(this.id)">
```

以下写法的this对象，都指向全局对象。

```javascript
// JavaScript代码
element.onclick = function (){ doSomething() };
element.setAttribute('onclick', 'doSomething()');

// HTML代码
<element onclick="doSomething()">
```

## 事件的传播

### 传播的三个阶段

当一个事件发生以后，它会在不同的DOM节点之间传播（propagation）。这种传播分成三个阶段：

- **第一阶段**：从window对象传导到目标节点，称为“捕获阶段”（capture phase）。

- **第二阶段**：在目标节点上触发，称为“目标阶段”（target phase）。

- **第三阶段**：从目标节点传导回window对象，称为“冒泡阶段”（bubbling phase）。

这种三阶段的传播模型，会使得一个事件在多个节点上触发。比如，假设div节点之中嵌套一个p节点。

```html
<div>
  <p>Click Me</p>
</div>
```

如果对这两个节点的click事件都设定监听函数，则click事件会被触发四次。

```javascript
var phases = {
  1: 'capture',
  2: 'target',
  3: 'bubble'
};

var div = document.querySelector('div');
var p = document.querySelector('p');

div.addEventListener('click', callback, true);
p.addEventListener('click', callback, true);
div.addEventListener('click', callback, false);
p.addEventListener('click', callback, false);

function callback(event) {
  var tag = event.currentTarget.tagName;
  var phase = phases[event.eventPhase];
  console.log("Tag: '" + tag + "'. EventPhase: '" + phase + "'");
}

// 点击以后的结果
// Tag: 'DIV'. EventPhase: 'capture'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'DIV'. EventPhase: 'bubble'
```

上面代码表示，click事件被触发了四次：p节点的捕获阶段和冒泡阶段各1次，div节点的捕获阶段和冒泡阶段各1次。

1. 捕获阶段：事件从div向p传播时，触发div的click事件；
2. 目标阶段：事件从div到达p时，触发p的click事件；
3. 目标阶段：事件离开p时，触发p的click事件；
4. 冒泡阶段：事件从p传回div时，再次触发div的click事件。

注意，用户点击网页的时候，浏览器总是假定click事件的目标节点，就是点击位置的嵌套最深的那个节点（嵌套在div节点的p节点）。

事件传播的最上层对象是window，接着依次是document，html（document.documentElement）和body（document.dody）。也就是说，如果body元素中有一个div元素，点击该元素。事件的传播顺序，在捕获阶段依次为window、document、html、body、div，在冒泡阶段依次为div、body、html、document、window。

### 事件的代理

由于事件会在冒泡阶段向上传播到父节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的代理（delegation）。

```javascript
var ul = document.querySelector('ul');

ul.addEventListener('click', function(event) {
  if (event.target.tagName.toLowerCase() === 'li') {
    // some code
  }
});
```

上面代码的click事件的监听函数定义在ul节点，但是实际上，它处理的是子节点li的click事件。这样做的好处是，只要定义一个监听函数，就能处理多个子节点的事件，而且以后再添加子节点，监听函数依然有效。

如果希望事件到某个节点为止，不再传播，可以使用事件对象的stopPropagation方法。

```javascript
p.addEventListener('click', function(event) {
  event.stopPropagation();
});
```

使用上面的代码以后，click事件在冒泡阶段到达p节点以后，就不再向上（父节点的方向）传播了。

但是，stopPropagation方法不会阻止p节点上的其他click事件的监听函数。如果想要不再触发那些监听函数，可以使用stopImmediatePropagation方法。

```javascript
p.addEventListener('click', function(event) {
 event.stopImmediatePropagation();
});

p.addEventListener('click', function(event) {
 // 不会被触发
});
```

## Event对象

事件发生以后，会生成一个事件对象，作为参数传给监听函数。浏览器原生提供一个Event对象，所有的事件都是这个对象的实例，或者说继承了`Event.prototype`对象。

Event对象本身就是一个构造函数，可以用来生成新的实例。

```javascript
event = new Event(typeArg, eventInit);
```

Event构造函数接受两个参数。第一个参数是字符串，表示事件的名称；第二个参数是一个对象，表示事件对象的配置。该参数可以有以下两个属性。

- bubbles：布尔值，可选，默认为false，表示事件对象是否冒泡。

- cancelable：布尔值，可选，默认为false，表示事件是否可以被取消。

```javascript
var ev = new Event("look", {"bubbles":true, "cancelable":false});
document.dispatchEvent(ev);
```

上面代码新建一个look事件实例，然后使用dispatchEvent方法触发该事件。

IE8及以下版本，事件对象不作为参数传递，而是通过window对象的event属性读取，并且事件对象的target属性叫做srcElement属性。所以，以前获取事件信息，往往要写成下面这样。

```javascript
function myEventHandler(event) {
  var actualEvent = event || window.event;
  var actualTarget = actualEvent.target || actualEvent.srcElement;
  // ...
}
```

上面的代码只是为了说明以前的程序为什么这样写，在新代码中，这样的写法不应该再用了。

以下介绍Event实例的属性和方法。

### bubbles，eventPhase

以下属性与事件的阶段有关。

**（1）bubbles**

bubbles属性返回一个布尔值，表示当前事件是否会冒泡。该属性为只读属性，只能在新建事件时改变。除非显式声明，Event构造函数生成的事件，默认是不冒泡的。

```javascript
function goInput(e) {
  if (!e.bubbles) {
    passItOn(e);
  } else {
    doOutput(e);
  }
}
```

上面代码根据事件是否冒泡，调用不同的函数。

**（2）eventPhase**

eventPhase属性返回一个整数值，表示事件目前所处的节点。

```javascript
var phase = event.eventPhase;
```

- 0，事件目前没有发生。
- 1，事件目前处于捕获阶段，即处于从祖先节点向目标节点的传播过程中。该过程是从Window对象到Document节点，再到HTMLHtmlElement节点，直到目标节点的父节点为止。
- 2，事件到达目标节点，即target属性指向的那个节点。
- 3，事件处于冒泡阶段，即处于从目标节点向祖先节点的反向传播过程中。该过程是从父节点一直到Window对象。只有bubbles属性为true时，这个阶段才可能发生。

### cancelable，defaultPrevented

以下属性与事件的默认行为有关。

**（1）cancelable**

cancelable属性返回一个布尔值，表示事件是否可以取消。该属性为只读属性，只能在新建事件时改变。除非显式声明，Event构造函数生成的事件，默认是不可以取消的。

```javascript
var bool = event.cancelable;
```

如果要取消某个事件，需要在这个事件上面调用preventDefault方法，这会阻止浏览器对某种事件部署的默认行为。

**（2）defaultPrevented**

defaultPrevented属性返回一个布尔值，表示该事件是否调用过preventDefault方法。

```javascript
if (e.defaultPrevented) {
  // ...
}
```

### currentTarget，target

以下属性与事件的目标节点有关。

**（1）currentTarget**

currentTarget属性返回事件当前所在的节点，即正在执行的监听函数所绑定的那个节点。作为比较，target属性返回事件发生的节点。如果监听函数在捕获阶段和冒泡阶段触发，那么这两个属性返回的值是不一样的。

```javascript
function hide(e){
  console.log(this === e.currentTarget);  // true
  e.currentTarget.style.visibility = "hidden";
}

para.addEventListener('click', hide, false);
```

上面代码中，点击para节点，该节点会不可见。另外，在监听函数中，currentTarget属性实际上等同于this对象。

**（2）target**

target属性返回触发事件的那个节点，即事件最初发生的节点。如果监听函数不在该节点触发，那么它与currentTarget属性返回的值是不一样的。

```javascript
function hide(e){
  console.log(this === e.target);  // 有可能不是true
  e.target.style.visibility = "hidden";
}

// HTML代码为
// <p id="para">Hello <em>World</em></p>
para.addEventListener('click', hide, false);
```

上面代码中，如果在para节点的em子节点上面点击，则`e.target`指向em子节点，导致em子节点（即World部分）会不可见，且输出false。

在IE6—IE8之中，该属性的名字不是target，而是srcElement，因此经常可以看到下面这样的代码。

```javascript
function hide(e) {
  var target = e.target || e.srcElement;
  target.style.visibility = 'hidden';
}
```

### type，detail，timeStamp，isTrusted

以下属性与事件对象的其他信息相关。

**（1）type**

type属性返回一个字符串，表示事件类型，具体的值同addEventListener方法和removeEventListener方法的第一个参数一致，大小写不敏感。

```javascript
var string = event.type;
```

**（2）detail**

detail属性返回一个数值，表示事件的某种信息。具体含义与事件类型有关，对于鼠标事件，表示鼠标按键在某个位置按下的次数，比如对于dblclick事件，detail属性的值总是2。

```javascript
function giveDetails(e) {
  this.textContent = e.detail;
}

el.onclick = giveDetails;
```

**（3）timeStamp**

`timeStamp`属性返回一个毫秒时间戳，表示事件发生的时间。

```javascript
var number = event.timeStamp;
```

Chrome在49版以前，这个属性返回的是一个整数，单位是毫秒（millisecond），表示从Unix纪元开始的时间戳。从49版开始，该属性返回的是一个高精度时间戳，也就是说，毫秒之后还带三位小数，精确到微秒。并且，这个值不再从Unix纪元开始计算，而是从`PerformanceTiming.navigationStart`开始计算，即表示距离用户导航至该网页的时间。如果想将这个值转为Unix纪元时间戳，就要计算`event.timeStamp + performance.timing.navigationStart`。

下面是一个计算鼠标移动速度的例子，显示每秒移动的像素数量。

```javascript
var previousX;
var previousY;
var previousT;

window.addEventListener('mousemove', function(event) {
  if (!(previousX === undefined ||
        previousY === undefined ||
        previousT === undefined)) {
    var deltaX = event.screenX - previousX;
    var deltaY = event.screenY - previousY;
    var deltaD = Math.sqrt(Math.pow(deltaX, 2) + Math.pow(deltaY, 2));

    var deltaT = event.timeStamp - previousT;
    console.log(deltaD / deltaT * 1000);
  }

  previousX = event.screenX;
  previousY = event.screenY;
  previousT = event.timeStamp;
});
```

**（4）isTrusted**

isTrusted属性返回一个布尔值，表示该事件是否可以信任。

```javascript
var bool = event.isTrusted;
```

Firefox浏览器中，用户触发的事件会返回true，脚本触发的事件返回false；IE浏览器中，除了使用createEvent方法生成的事件，所有其他事件都返回true；Chrome浏览器不支持该属性。

### preventDefault()

preventDefault方法取消浏览器对当前事件的默认行为，比如点击链接后，浏览器跳转到指定页面，或者按一下空格键，页面向下滚动一段距离。该方法生效的前提是，事件的cancelable属性为true，如果为false，则调用该方法没有任何效果。

该方法不会阻止事件的进一步传播（stopPropagation方法可用于这个目的）。只要在事件的传播过程中（捕获阶段、目标阶段、冒泡阶段皆可），使用了preventDefault方法，该事件的默认方法就不会执行。

```javascript
// HTML代码为
// <input type="checkbox" id="my-checkbox" />

var cb = document.getElementById('my-checkbox');

cb.addEventListener(
  'click',
  function (e){ e.preventDefault(); },
  false
);
```

上面代码为点击单选框的事件，设置监听函数，取消默认行为。由于浏览器的默认行为是选中单选框，所以这段代码会导致无法选中单选框。

利用这个方法，可以为文本输入框设置校验条件。如果用户的输入不符合条件，就无法将字符输入文本框。

```javascript
function checkName(e) {
  if (e.charCode < 97 || e.charCode > 122) {
    e.preventDefault();
  }
}
```

上面函数设为文本框的keypress监听函数后，将只能输入小写字母，否则输入事件的默认事件（写入文本框）将被取消。

如果监听函数最后返回布尔值false（即return false），浏览器也不会触发默认行为，


