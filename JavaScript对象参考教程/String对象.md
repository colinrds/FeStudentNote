## 概述
String对象是JavaScript原生提供的三个包装对象之一，用来生成字符串的包装对象。
```
var s1 = 'abc';
var s2 = new String('abc');

typeof s1 // "string"
typeof s2 // "object"

s2.valueOf() // "abc"
```
上面代码中，变量s1是字符串，s2是对象。由于s2是对象，所以有自己的方法，valueOf方法返回的就是它所包装的那个字符串。

实际上，字符串的包装对象是一个类似数组的对象（即很像数组，但是实质上不是数组）。

```
new String("abc")
// String {0: "a", 1: "b", 2: "c", length: 3}
```
除了用作构造函数，String对象还可以当作工具方法使用，将任意类型的值转为字符串。

```
String(true) // "true"
String(5) // "5"
```
上面代码将布尔值ture和数值5，分别转换为字符串。

## String.fromCharCode()
String对象提供的静态方法（即定义在对象本身，而不是定义在对象实例的方法），主要是fromCharCode()。该方法的参数是一系列Unicode码点，返回对应的字符串。

```
String.fromCharCode(104, 101, 108, 108, 111)
// "hello"
```
上面代码中，fromCharCode方法接受5个Unicode码点作为参数，返回它们组成的字符串。

注意，该方法不支持Unicode码点大于0xFFFF的字符，即传入的参数不能大于0xFFFF。

```
String.fromCharCode(0x20BB7)
// "ஷ"
```
上面代码返回字符的编号是0x0BB7，而不是0x20BB7。它的根本原因在于，码点大于0xFFFF的字符占用四个字节，而JavaScript只支持两个字节的字符。这种情况下，必须把0x20BB7拆成两个字符表示。

```
String.fromCharCode(0xD842, 0xDFB7)
// "𠮷"
```
上面代码中，0x20BB7拆成两个字符0xD842和0xDFB7（即两个两字节字符，合成一个四字节字符），就能得到正确的结果。码点大于0xFFFF的字符的四字节表示法，由UTF-16编码方法决定。

## 实例对象的属性和方法

### length属性
length属性返回字符串的长度。

```
'abc'.length // 3
```
### charAt()
charAt方法返回指定位置的字符，参数是从0开始编号的位置。

```
var s = new String('abc');

s.charAt(1) // "b"
s.charAt(s.length - 1) // "c"
```

这个方法完全可以用数组下标替代。

```
'abc'.charAt(1) // "b"
'abc'[1] // "b"
如果参数为负数，或大于等于字符串的长度，charAt返回空字符串。

'abc'.charAt(-1) // ""
'abc'.charAt(3) // ""
```

### charCodeAt()
charCodeAt方法返回给定位置字符的Unicode码点（十进制表示），相当于String.fromCharCode()的逆操作。

```
'abc'.charCodeAt(1) // 98
```
上面代码中，abc的1号位置的字符是b，它的Unicode码点是98。

如果没有任何参数，charCodeAt返回首字符的Unicode码点。

```
'abc'.charCodeAt() // 97
```
上面代码中，首字符a的Unicode编号是97。

需要注意的是，charCodeAt方法返回的Unicode码点不大于65536（0xFFFF），也就是说，只返回两个字节的字符的码点。如果遇到Unicode码点大于65536的字符，必需连续使用两次charCodeAt，不仅读入charCodeAt(i)，还要读入charCodeAt(i+1)，将两个16字节放在一起，才能得到准确的字符。

如果参数为负数，或大于等于字符串的长度，charCodeAt返回NaN。

### concat()
concat方法用于连接两个字符串，返回一个新字符串，不改变原字符串。

```
var s1 = 'abc';
var s2 = 'def';

s1.concat(s2) // "abcdef"
s1 // "abc"
```

该方法可以接受多个参数。

```
'a'.concat('b', 'c') // "abc"
```

如果参数不是字符串，concat方法会将其先转为字符串，然后再连接。

```
var one = 1;
var two = 2;
var three = '3';

''.concat(one, two, three) // "123"
one + two + three // "33"
```

上面代码中，concat方法将参数先转成字符串再连接，所以返回的是一个三个字符的字符串。作为对比，加号运算符在两个运算数都是数值时，不会转换类型，所以返回的是一个两个字符的字符串。

### slice()
slice方法用于从原字符串取出子字符串并返回，不改变原字符串。

它的第一个参数是子字符串的开始位置，第二个参数是子字符串的结束位置（不含该位置）。

```
'JavaScript'.slice(0, 4) // "Java"
```
如果省略第二个参数，则表示子字符串一直到原字符串结束。

```
'JavaScript'.slice(4) // "Script"
```
如果参数是负值，表示从结尾开始倒数计算的位置，即该负值加上字符串长度。

```
'JavaScript'.slice(-6) // "Script"
'JavaScript'.slice(0, -6) // "Java"
'JavaScript'.slice(-2, -1) // "p"
```
如果第一个参数大于第二个参数，slice方法返回一个空字符串。

```
'JavaScript'.slice(2, 1) // ""
```

### substring()
substring方法用于从原字符串取出子字符串并返回，不改变原字符串。它与slice作用相同，但有一些奇怪的规则，因此不建议使用这个方法，优先使用slice。

substring方法的第一个参数表示子字符串的开始位置，第二个位置表示结束位置。

```
'JavaScript'.substring(0, 4) // "Java"
```
如果省略第二个参数，则表示子字符串一直到原字符串的结束。

```
'JavaScript'.substring(4) // "Script"
```
如果第二个参数大于第一个参数，substring方法会自动更换两个参数的位置。

```
'JavaScript'.substring(10, 4) // "Script"
// 等同于
'JavaScript'.substring(4, 10) // "Script"
```
上面代码中，调换substring方法的两个参数，都得到同样的结果。

如果参数是负数，substring方法会自动将负数转为0。

```
'Javascript'.substring(-3) // "JavaScript"
'JavaScript'.substring(4, -3) // "Java"
```

上面代码的第二个例子，参数-3会自动变成0，等同于'JavaScript'.substring(4, 0)。由于第二个参数小于第一个参数，会自动互换位置，所以返回Java。

### substr()
substr方法用于从原字符串取出子字符串并返回，不改变原字符串。

substr方法的第一个参数是子字符串的开始位置，第二个参数是子字符串的长度。

```
'JavaScript'.substr(4, 6) // "Script"
```
如果省略第二个参数，则表示子字符串一直到原字符串的结束。

```
'JavaScript'.substr(4) // "Script"
```
如果第一个参数是负数，表示倒数计算的字符位置。如果第二个参数是负数，将被自动转为0，因此会返回空字符串。

```
'JavaScript'.substr(-6) // "Script"
'JavaScript'.substr(4, -1) // ""
```
上面代码的第二个例子，由于参数-1自动转为0，表示子字符串长度为0，所以返回空字符串。

### indexOf()，lastIndexOf()

这两个方法用于确定一个字符串在另一个字符串中的位置，都返回一个整数，表示匹配开始的位置。如果返回-1，就表示不匹配。两者的区别在于，indexOf从字符串头部开始匹配，lastIndexOf从尾部开始匹配。

```
'hello world'.indexOf('o') // 4
'JavaScript'.indexOf('script') // -1

'hello world'.lastIndexOf('o') // 7
```
它们还可以接受第二个参数，对于indexOf方法，第二个参数表示从该位置开始向后匹配；对于lastIndexOf，第二个参数表示从该位置起向前匹配。

```
'hello world'.indexOf('o', 6) // 7
'hello world'.lastIndexOf('o', 6) // 4
```

### trim()
trim方法用于去除字符串两端的空格，返回一个新字符串，不改变原字符串。

```
'  hello world  '.trim()
// "hello world"
```

该方法去除的不仅是空格，还包括制表符（\t、\v）、换行符（\n）和回车符（\r）。

```
'\r\nabc \t'.trim() // 'abc'
```

### toLowerCase()，toUpperCase()

toLowerCase方法用于将一个字符串全部转为小写，toUpperCase则是全部转为大写。它们都返回一个新字符串，不改变原字符串。

```
'Hello World'.toLowerCase()
// "hello world"

'Hello World'.toUpperCase()
// "HELLO WORLD"
```

这个方法也可以将布尔值或数组转为大写字符串，但是需要通过call方法使用。

```
String.prototype.toUpperCase.call(true)
// 'TRUE'
String.prototype.toUpperCase.call(['a', 'b', 'c'])
// 'A,B,C'
```

### localeCompare()
localeCompare方法用于比较两个字符串。它返回一个整数，如果小于0，表示第一个字符串小于第二个字符串；如果等于0，表示两者相等；如果大于0，表示第一个字符串大于第二个字符串。

```
'apple'.localeCompare('banana')
// -1

'apple'.localeCompare('apple')
// 0
```

该方法的最大特点，就是会考虑自然语言的顺序。举例来说，正常情况下，大写的英文字母小于小写字母。

```
'B' > 'a' // false
```

上面代码中，字母B小于字母a。这是因为JavaScript采用的是Unicode码点比较，B的码点是66，而a的码点是97。

但是，localeCompare方法会考虑自然语言的排序情况，将B排在a的前面。

```
'B'.localeCompare('a') // 1
```

上面代码中，localeCompare方法返回整数1（也有可能返回其他正整数），表示B较大。

### match()
match方法用于确定原字符串是否匹配某个子字符串，返回一个数组，成员为匹配的第一个字符串。如果没有找到匹配，则返回null。

```
'cat, bat, sat, fat'.match('at') // ["at"]
'cat, bat, sat, fat'.match('xt') // null
```

返回数组还有index属性和input属性，分别表示匹配字符串开始的位置和原始字符串。

```
var matches = 'cat, bat, sat, fat'.match('at');
matches.index // 1
matches.input // "cat, bat, sat, fat"
```

match方法还可以使用正则表达式作为参数，详见《正则表达式》一节。

### search()
search方法的用法等同于match，但是返回值为匹配的第一个位置。如果没有找到匹配，则返回-1。

```
'cat, bat, sat, fat'.search('at') // 1
```
search方法还可以使用正则表达式作为参数，详见《正则表达式》一节。

### replace()
replace方法用于替换匹配的子字符串，一般情况下只替换第一个匹配（除非使用带有g修饰符的正则表达式）。

```
'aaa'.replace('a', 'b') // "baa"
```
replace方法还可以使用正则表达式作为参数，详见《正则表达式》一节。

### split()
split方法按照给定规则分割字符串，返回一个由分割出来的子字符串组成的数组。

```
'a|b|c'.split('|') // ["a", "b", "c"]
```
如果分割规则为空字符串，则返回数组的成员是原字符串的每一个字符。

```
'a|b|c'.split('') // ["a", "|", "b", "|", "c"]
```

如果省略参数，则返回数组的唯一成员就是原字符串。

```
'a|b|c'.split() // ["a|b|c"]
```

如果满足分割规则的两个部分紧邻着（即中间没有其他字符），则返回数组之中会有一个空字符串。

```
'a||c'.split('|') // ['a', '', 'c']
```

如果满足分割规则的部分处于字符串的开头或结尾（即它的前面或后面没有其他字符），则返回数组的第一个或最后一个成员是一个空字符串。

```
'|b|c'.split('|') // ["", "b", "c"]
'a|b|'.split('|') // ["a", "b", ""]
```

split方法还可以接受第二个参数，限定返回数组的最大成员数。

```
'a|b|c'.split('|', 0) // []
'a|b|c'.split('|', 1) // ["a"]
'a|b|c'.split('|', 2) // ["a", "b"]
'a|b|c'.split('|', 3) // ["a", "b", "c"]
'a|b|c'.split('|', 4) // ["a", "b", "c"]
```

上面代码中，split方法的第二个参数，决定了返回数组的成员数。