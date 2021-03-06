# ES6标准全整理

## let、const命令和块级作用域

### let命令

let命令声明的变量只在其所在的代码块中有效。

#### 不存在变量提升

变量提升的概念存在于var命令，指的是变量在声明之前可使用，并且值为undefined，这并不符合一般编程逻辑，即先声明后使用。

在ES5中，为了减少运行错误，强调养成良好的编程习惯，不使用未声明的变量，避免出现意料之外的行为，但不使用不代表不可用，这并没有从根源上解决问题。

在ES6中，let命令纠正了这一对象，如果使用未声明的变量，编译将会报错。

#### 暂时性死区

先看下面这个例子：

```javascript
var x = "hello world"
if (true) {
    console.log(x)//reference error
    let x = "goodbye helloworld"
}
```

在if代码块中，let命令声明了一个x变量，在声明之前使用x变量时，报错。从代码块开始到let声明变量x之前，这个区域是变量x的暂时性死区，即使这个变量在外部已经声明。

##### 注意

暂时性死区导致typeof不再是100%安全的操作。反而，如果一个变量没有被声明，使用typeof不会报错。
因此良好的编程习惯一定要保持：变量一定要在声明之后使用。

#### 不允许重复声明

在相同作用域内不能使用let重复声明变量。

#### 特殊的for循环

```javascript
for (let i = 0; i <= 10; i++) {
    let i = "123"
    console.log(i)
}
```

结果输出10个"123"字符串。

在这个for循环中，循环变量是父作用域，循环内部是子作用域，两个i不在同一个作用域内，因此互不影响。

#### 全局变量与顶层对象无关

在JavaScript中，顶层对象的属性等价于全局变量，var声明的全局变量也能通过顶层属性访问得到，而且顶层对象根据环境的不同而异，比如浏览器中的window对象。这带来了几个困扰：

* 无法在编译时提示变量未声明的错误，因为这个变量可能为顶层对象的属性。
* 可能在不知不觉中添加了全局变量。
* 顶层对象的属性到处可读写，不利于模式化编程。
* 顶层对象不应该包含有实体含义，比如window对象包含了窗口的实体含义。

在ES6中，let声明的全局变量不属于顶层对象的属性(包括const、class)，由此与顶层对象隔离开。但是var和function命令声明的变量仍然属于顶层对象的属性，以保持兼容性。

###const命令

const命令与let命令相似，区别是变量为只读模式，一旦声明之后值不可以改变。

const只读的本质不是变量值不得改动，而是指向的内存地址不得改动。

* 简单数据类型：值就保存在指向的内存地址中。

* 复合数据类型：内存地址保存的是指针，至于指针指向的数据结构，const不做任何保证，因此操作时必须小心。

如果希望对象下属性的值也只读，可以使用`object.freeze`方法冻结对象。

```javascript
const obj = Object.freeze({foo: "123"})
obj.foo = "abc"
obj.foo//"123"
```

常规模式下，对属性赋值不起任何作用；严格模式下，对属性赋值会报错。

想要彻底冻结，如果对象的属性值的数据类型也是对象，那么也需要冻结该属性的值。

### 块级作用域

在ES5中，只有全局作用域和函数作用域，并没有块级作用域，导致很多场景不合理。比如：

* 内层变量覆盖外层变量
* 计数循环变量泄漏成为全局变量

没有块级作用域的情况下，我们常使用了IIFE写法模拟块级作用域，来避免变量污染。

```javascript
(function() {
    //...
}());
```

在ES6中，有了let和const命令，声明的变量只在其代码块中有效，这里就有了块级作用域。

## 解构赋值

**解构**指的是通过**模式匹配**从数组或对象中提取值。当等号两边的模式相同，左边的变量会被赋予右边对应的值。

**不完全解构**：等号左边的模式只匹配一部分等号右边的模式。

### 不同数据类型的解构赋值

#### 数组

按照顺序匹配值。

```javascript
let [a, b, c] = [1, 2, 3]
let [a, [b1, b2], c] = [1, [2.1, 2.2], 3]
```

只要某种数据结构具有Iterator接口，都可以采用数组形式进行解构赋值。

#### 对象

按照属性名匹配值。

```javascript
let {foo} = {foo: "123"}
```

匹配对象中的foo属性，并且将属性值赋值给变量foo。因此，上面解构的最终变量名与属性名一致。

如果希望最终变量名不与属性名一致，须写成：

```javascript
let {foo: baz} = {foo: "123"}
```

其中foo指的是匹配模式，baz指的是变量名，解构赋值的内部机制是先找到同名属性，再赋值给变量。所以：

```javascript
let {foo} = {foo: "123"}
//等同于以下表达式的简写
let {foo: foo} = {foo: "123"}
```

也可用于嵌套模式

```javascript
let {p: {x: baz}} = {p: {x: 1, y: 2}}
baz//1
```

实际上，数组是特殊的对象，因此数组也可以使用对象形式进行解构赋值，匹配的属性名即数组索引。

```javascript
let {0: first, 1: second, length: len} = [1, 2, 3]
first//1
length//3
```

#### 其他数据类型

如果等号右边是其他数据类型，那么在进行解构之前，会先将值转换为对象。但undefined和null无法转换成对象，所以当右边值为undefined或null时，解构赋值会报错。

##### 字符串

转换成一个类似数组的对象。

```javascript
let {length: len} = "helloworld";
len//10
```

##### 数值

```javascript
let {toString: s} = 12;
s === Number.prototype.toString//true
```

等等。

#### 注意

以上都是声明赋值，已声明的变量也可用于解构赋值。

```javascript
let x
[x] = [1]
x//1
({foo: x} = {foo: "123"})
x//"123"
```

这里要注意，没有声明命令的对象形式的解构赋值，行首是以大括号开始的。JS引擎会将行首为大括号的`{x}`理解成代码块，从而引发语法错误。所以使用圆括号包起来，不让大括号成为行首。

### 允许指定默认值

```javascript
let [x = 1] = [];
x//1
let {y: z = 1} = {}
z//1
```

ES6内部使用严格相等运算符(===)判断一个位置是否有值，如果值===undefined，那么使用默认值。

```javascript
let [x = 1] = [undefined]
x//1
let [y = 1] = [null]
y//null
```

如果默认值是一个表达式，那么这个表达式是惰性求值，即只有用到才求值。

```javascript
let counter = 0;
function getVal() {
    return counter++;
}
let [x = getVal()] = [1];
counter//0
```

上面的例子中，x匹配得到值，因此getVal()没有执行，counter值未变。

### 用途

#### 变量值交换

```javascript
let x = 1, y = 2
[x, y] = [y, x]
x//2
y//1
```

#### 从函数返回多个值

```javascript
function getVal() {
    return [1, 2]
}
let [a, b] = getVal();
```

#### 函数参数的定义

数组有序传入多个参数

```javascript
function setVal([a, b, c]) {}
setVal([1, 2, 3])
```

对象无序传入多个参数。当函数参数多时，将参数集合到对象中传入函数，由函数解构获值，可免去按照顺序传入多参数的烦恼。

```javascript
function setVal({a, b, c}) {}
setVal({b: 2, a: 1, c: 3})
```

#### 提取JSON数据

#### 函数参数指定默认值

## 字符串的拓展

### unicode字符

ES6加强了对unicode字符的支持。

#### 加强unicode字符的表示能力

众所周知，JavaScript允许使用`\uxxxx`的形式来表示字符，其中`xxxx`表示的是字符码点，码点范围区间是`0x0000`-`oxFFFF`，超出这个范围的字符，需要使用两个双字节的形式来表示。

在ES5中，如果`\u`后面的码点超过`0xFFFF`，比如`\u20BB7`，那么它会被JavaScript理解为两个字符，即`\u20BB`和7。

ES6解决了这个问题，将码点放在大括号中，比如`\u{20BB7}`，这个字符能被正确解读，即使它的大小超过1个双字节。

### 字符串方法

JS使用UTF-16格式来存储字符串，每个字符固定是2个字节大小。但是对于需要使用4个字节存储的字符，JS做了错误的判断：4个字节大小的字符，在字符串中体现的长度为2。

#### codePointAt

在ES5中，使用charCodeAt方法，能够准确范围2个字节存储的字节的码点，但对于4个字节存储的字符，只能根据JS误解的字符长度，返回前两个字节或后两个字节的码点。

在ES6中，提供了一种新的方法codePointAt方法，参数为字符串索引，能够准确识别4字节的字符，返回对应的码点。但对于JS，它仍然会将4字节的字符长度理解错误。

* 当索引指向4字节字符的前半部分时，codePointAt方法会准确识别出4字节字符的码点并且返回。
* 当索引指向4字节字符的后半部分时，此时codePointAt方法同charCodeAt一样，返回4字节字符后2个字节的码点。

可使用codePointAt方法来识别一个字符是2字节还是4字节。

```javascript
function is32bit(c) {
    return c.codePointAt(0) > 0xFFFF
}
```

#### fromCodePoint

与codePointAt相反，codePointAt是将字符转为码点，fromCodePoint则是将码点转为字符。

在ES5中，可使用String.fromCharCode从码点获取字符，同样的，它不支持获取4字节字符。

在ES6中，可使用String.fromCodePoint准确获取到2字节或4字节字符。

#### includes、startsWith和endsWith

在ES5中，字符串只有indexOf方法可用于判断一个字符串是否包含在另一个字符串中，如果包含，则返回匹配的索引位置，如果不包含，则返回-1。

在ES6中，提供了3种新的方法。

`includes(str, index)`从第index个字符开始到结束，是否能匹配到与str相同的字符，并且返回Boolean值。

`startsWith(str, index)`从第index个字符开始到结束这段字符区间，是否以str字符开头的，并返回Boolean值。

`endswith(str, index)`从字符串开始到第index个字符(不指定index，默认到结尾)的这段字符区间，是否以str字符结尾的，并返回Boolean值。

#### repeat

`repeat(n)`返回一个将原字符串重复n次的新字符串。

如果n是小数，舍去小数部分，取整。

如果n是0～-1，等同于`repeat(0)`。

如果n是NaN，等同于`repeat(0)`。

如果n是小于等于-1的负数或是个无限数，报错。

#### padStart、padEnd

这两个方法用于补全字符串，传入参数格式为`(length, str)`。

length：如果字符串长度不超过length，要进行字符串补全。如果字符串长度等于甚至超过length，直接返回原字符串。

str：用于补全的字符串，如果没有传str，默认使用空格进行补全。

这两个方法常用于**为数值补全指定值**。

#### for...of

上边说到，针对4字节字符，JS在字符串中会将这个字符的长度理解为2。如何才能准确将字符串中的字符完整地遍历？

ES6给字符串对象添加了遍历器接口，使得字符串可通过for...of进行循环，并且能够识别4字节字符。

### 模板字符串

普通输出模板：

`"My name is " + name + "and I am " + age + "years old."`

如果变量较多，这种写法十分烦琐。

模板字符串：增强版的字符串。

```javascript
`My name is ${name} and I am ${age} years old.`
```

用反引号标识，可用于普通字符串，也可用于多行字符串，还可在字符串种嵌入变量或表达式。

#### 标签模板

这里的标签指的是函数，模板字符串跟在标签后面，表格这个函数将被调用来处理这个模板字符串。

```javascript
alert`123`
//等同于
alert(123)
```

如果标签后的模板字符串带有变量，那就不是简单的调用了。这个模板字符串会被处理成多个参数然后被函数调用。

第一个参数是：字符串根据${}部分分隔成字符串数组。

第二及以后的个参数：字符串中{}内的变量或表达式结果。



## 正则表达式

在ES5中，使用RegExp构造函数生成正则表达式的方式有两种：

```javascript
//第一个参数是匹配字符串，第二个参数是正则修饰符
var regex = new RegExp("xyz", "i")
//拷贝原正则表达式
var regex1 = new RegExp(/xyz/i);
```

这两种方式传参格式不同，第二种方式不支持传参指定修饰符。

在ES6中，修改了这种方式，在第二种方式的基础上增加了修饰符参数。

```javascript
var regex2 = new RegExp(/xyz/, 'i')
```

如果原正则表达式带有修饰符，则会替换成新传入的修饰符。

### 查看修饰符

可使用flags属性查看正则表达式当前的修饰符。

```javascript
/a+/ig.flags //"gi"
```

### ES6新增的修饰符

#### u修饰符(unicode模式)

用于纠正JS无法准确解读4字节字符的问题。

1. 准确解读4字节字符为单个字符
   * 比如`\uD83B\uDC2A`这个单字符，`/^\uD83B/u.test(\uD83B\uDC2A)`匹配失败。
   * 点字符，常被用于匹配出换行符之外的单字符，但却无法准确匹配到4字节的单字符，使用unicode模式能使得点字符正确识别4字节字符。
   * 预定义模式\S，用于匹配所有不是空格的单字符，在unicode模式下能准确识别4字节字符。
2. 支持使用unicode大括号表示法。在原有的正则表达式中，大括号用于表示量词，不标明unicode模式会导致unicode字符的大括号解读错误。

#### y修饰符(粘连修饰符)

与g修饰符相同，均为全局匹配。

与g修饰符的区别在于，每次匹配，g修饰符只要在剩余位置中找到匹配结果并返回即可，但是y修饰符会确保从第一个位置开始进行头部匹配，这就是粘连的作用。

```javascript
var str = "_yyy"
/y+/g.exec(str) //"yyy"
/y+/y.exec(str) //null
```

可使用sticky属性判断正则表达式是否设置粘连修饰符。

```javascript
var regex = new RegExp(/a+/y)
regex.sticky //true
```

## 数值拓展

### 二进制八进制表示法

ES5中不同进制的表示法：

二进制：没找着...

八进制：`0123 === 83，`以`0`为前缀，如果后面数字均小于8，该数值会被视为八进制，但如果存在8或9，该数值会被视为十进制，这种判断模式很容易混乱。因此，严格模式下不允许使用`0`作为前缀表示八进制。

十六进制：`0x00FF ===255 `，常见的颜色代码就是用十六进制表示的。

ES6提出了二进制和八进制新的表示法：

二进制：`0b10101010 === 170`，以`0b`(`0B`)作为二进制写法的前缀。

八进制：`0o123 === 83`，以`0o`(`0O`)作为八进制写法的前缀，为了兼容旧代码，浏览器还是继续支持`0`作为前缀的表示法。

### Number对象拓展

#### isFinite()

检查数值是否为有限值，即不为Infinity。

```javascript
Number.isFinite(1024) //true
Number.isFinite(Infinity) //false
Number.isFinite(NaN) //false
Number.isFinite("12") //false
Number.isFinite("12bb") //false
```

除了Infinity，参数类型不为数值也会返回false。

```javascript
isFinite("12") //true
isFinite("12bb") //false
```

与全局方法`isFinite`的区别在于：在判断参数之前，全局方法`isFinite`会调用`Number()`将参数转为数值，而`Number.isFinite`对于非数值的参数一律返回`false`。

#### isNaN()

检查数值是否为NaN。

```javascript
Number.isNaN(NaN) //true

Number.isNaN("NaN") //false
isNaN("NaN") //true

Number.isNaN("1212") //false
isNaN("1212") //false
```

与全局方法`isNaN`的区别在于：在判断参数之前，全局方法`isNaN`会调用`Number()`将参数转为数值，而`Number.isNaN`对于非`NaN`的参数一律返回`false`。

#### isInteger()

判断数值是否为整数。

```javascript
Number.isInteger(123) //true
Number.isInteger(100.0) //true
Number.isInteger(100.1) //false
Number.isInteger("12") //false
```

参数类型非数值，一律返回`false`。

```javascript
Number.isInteger(3.0000000000000002) // true
```

JS采用64位双精度格式存储数值，一旦精度值超过53位(1个隐藏位和52个有效位)，超过的部分直接忽略，此时可能出现数据误判问题。

#### parseInt()和parseFloat()

ES6将全局方法`parseInt`和`parseFloat`一直到`Number`对象上，行为保持一致。

```javascript
parseInt === Number.parseInt //true
parseFloat === Number.parseFloat //true
```

#### EPSILON

极小的常量，是JS能达到的最小精度，常用来设置JavsScript能够接受的误差范围。

#### 安全整数

一旦数值超出`-2^53`~`2^53`范围，就不能被准确表示。

##### MIN_SAFE_INTEGER和MAX_SAFE_INTEGER

表示能准确表示的数值范围的上下边界。

##### isSafeInteger()

检查数值是否在处于安全范围之内(`MIN_SAFE_INTEGER`~`MAX_SAFE_INTEGER`)

### Math对象拓展

#### trunc()

除去数值的小数部分。

```javascript
Math.trunc(1.23) //1
Math.trunc(1.9) //1

Math.trunc("1.23") //1
Math.trunc(true) //1
Math.trunc(false) //0

Math.trunc(NaN) //NaN
Math.trunc("hello") //NaN
Math.trunc() //NaN
```

若参数非数值，则先通过`Number()`进行转换。如果参数为空或参数转换结果为`NaN`，一律返回`NaN`。

#### sign()

判断数值是负数、0还是正数。

若参数非数值，则先通过`Number()`进行转换。

```javascript
Math.sign(-1) //-1
Math.sign(-0) //-0
Math.sign(0) //0
Math.sign(1) //1
Math.sign(NaN) //NaN
```

若参数非数值，则先通过`Number()`进行转换。如果参数为空或参数转换结果为`NaN`，一律返回`NaN`。

#### cbrt()

用于立方根计算。

```javascript
Math.cbrt(8) //2
```

若参数非数值，则先通过`Number()`进行转换。如果参数为空或参数转换结果为`NaN`，一律返回`NaN`。

#### clz32()

将数值转成32位无符号整数形式，并统计有多少个前导0。

```javascript
Math.clz32(0) //32
Math.clz32(1) //31
Math.clz32(0b01000000000000000000000000000000) //1
Math.clz32(1 << 10) //21
Math.clz32(3.2) //30
Math.clz32() //32
```

`clz32`这个函数名就来自”count leading zero bits in 32-bit binary representation of a number“的缩写。 

若参数为小数，则取整数部分进行前导0统计。

若参数非数值，则先通过`Number()`进行转换。如果参数为空或参数转换结果为`NaN`，一律返回32。

#### imul()

乘法计算。等同于`(a * b) | 0`，超出32位的部分溢出。

```javascript
Math.imul(-3, 3) //9
```

因为JavaScript有精度限制，所以很大的数的乘法，结果的低位数值往往是不准确的，使用`imul`方法可以返回准确的低位数值。

```javascript
Math.imul(0x7fffffff, 0x7fffffff) //1
```

#### fround()

返回一个数的32位单精度浮点数形式 。可用于将64位双精度浮点数转为32位单精度浮点数。

```javascript
Math.fround(2) //2
Math.fround(2 ** 24) //16777216
Math.fround(2 ** 24 + 1) //16777216

Math.fround(1.25) //1.25
Math.fround(NaN) //NaN
Math.fround(Infinity) //Infinity
```

单精度浮点数数值范围在`-2^24`~`2^24`之间，如果超过这个范围，就会丢失精度。

对于NaN和Infinity，则返回原值。

#### hypot()

计算所有数值平方和的平方根（两个值的话岂不就是勾股定理？）。

```javascript
Math.hypot(3, 4) //5
Math.hypot(3, "4") //5
Math.hypot(3, 4, 5) //7.0710678118654755
Math.hypot() //0
Math.hypot(NaN) //NaN
Math.hypot(3, "abc") //NaN
```

若参数非数值，则先通过`Number()`进行转换。如果参数为空， 返回0，如果参数转换结果为`NaN`，一律返回`NaN`。

#### expm1()

`Math.expm1(x)`等同于`Math.exp(x) - 1`。

#### log1p()

`Math.log1p(x)`等同于`Math.log(1 + x)`，x小于-1时返回`NaN`。

#### log2()和log10()

显而易见，不解释。

#### 指数运算符(**)

```javascript
2 ** 2 //4
2 ** 2 ** 3 //256
```

多指数运算符连用时，右结合计算。

##### 指数赋值运算符(**=)

```javascript
let a = 2
a **= 2
a //4
```

