
（转载文章，[原文地址](http://lib.csdn.net/article/javascript/58710?knId=507)）

典型的面向对象编程语言（比如C++和Java），存在“类”（class）这个概念。所谓“类”就是对象的模板，对象就是“类”的实例。但是JavaScript语言的对象体系，不是基于“类”的，而是基于构造函数（constructor）和原型链（prototype）。

以下的内容会分为如下细节：
1.对象的概念
2.构造函数
3.new 命令
  3.1基本原理
  3.2基本用法

### 1. 对象的概念
“面向对象编程”（Object Oriented Programming，缩写为OOP）是目前主流的编程范式。它的核心思想是将真实世界中各种复杂的关系，抽象为一个个对象，然后由对象之间的分工合作，完成对真实世界的模拟。

在JavaScript的重要数据类型-对象这篇文章中提到了js中对象的一些基本知识，比如说对象的创建， 对象的引用，对象的属性等等。如果没有掌握对象的基础知识，请移步JavaScript的重要数据类型-对象。

在有了对象的基础知识之后，对js中对象做一个简单的总结。如下：

a: 对象是单个实物的抽象。
b: 对象是一个容器，封装了‘属性’和‘方法’.

一本书、一辆汽车、一个人都可以是“对象”。当实物被抽象成“对象”，实物之间的关系就变成了“对象”之间的关系，从而就可以模拟现实情况，针对“对象”进行编程。

所谓属性，就是对象的一种状态；所谓方法，就是对象的一种行为。

比如说，可以把动物抽象为animal对象，属性记录的就是哪一种动物，以及该动物的大小和颜色等。方法方法表示该动物的某中行为（奔跑，猎食，交配，休息等等）.

### 2. 构造函数
‘面向对象编程’的第一步，就是要生成对象。而js中面向对象编程是基于构造函数（constructor）和原型链（prototype）的。

前面说过，“对象”是单个实物的抽象。通常需要一个模板，表示某一类实物的共同特征，然后“对象”根据这个模板生成。

js语言中使用构造函数（constructor）作为对象的模板。所谓构造函数，就是提供一个生成对象的模版，并描述对象的基本结构的函数。一个构造函数，可以生成多个对象，每个对象都有相同的结构。

看一下构造函数的基本结构。
```
const Keith = function() {
  this.height = 180;
};
// 两种写法相同。
function Keight() {
  this.height = 180;
}
```
上面代码中，`Keith`就是构造函数，它提供模板，用来生成对象实例。为了与普通函数区别，构造函数名字的第一个字母通常大写。

构造函数的三大特点：
- 构造函数的函数名的第一个字母通常大写。
- 函数体内使用`this`关键字，代表所要生成的对象实例。
- 生成对象的时候，必须使用new命令来调用构造函数。

### 3. new命令
#### 3.1 基本原理
`new`命令的作用，就是执行一个构造函数，并且返回一个对象实例。使用`new`命令时，它后面的函数调用就不是正常的调用，而是依次执行下面的步骤：
- 创建一个空对象，作为将要返回的对象实例
- 将空对象的原型指向构造函数的`prototype`属性
- 将空对象赋值给构造函数内部的`this`关键字
- 开始执行构造函数内部的代码
也就是说，构造函数内部，`this`指向的是一个新生成的对象，所有针对`this`的操作，都会发生在这个空对象上。构造函数之所谓构造函数，意思是这个函数的目的就是操作一个空对象（即`this`对象），将其构造为需要的样子。

以上是`new`命令的基本原理，这个很重要。以下会用具体实例来验证该原理的过程。

#### 3.2基本用法
`new`命令的作用，就是调用一个构造函数，并且返回一个对象实例。
```
function Keith() {
  this.height = 180;
}

const boy = new Keith();
console.log(boy.height);  // 180
```
上面代码中通过`new`命令，让构造函数`Keith`生成一个对象实例，并赋值给全局变量`boy`。这个新生成的对象实例，从构造函数`Keith`中继承了`height`属性。也就说明了这个对象实例是没有`height`属性的。在`new`命令执行时，就代表了新生成对象实例boy。`this.height`代表对象实例有一个`height`属性，它的值是180.

使用`new`命令时，根据需要，构造函数也可以接收参数。
```
function Person(name, height) {
  this.name = name;
  this.height = height;
}

const boy = new Person('Keith', 180);
console.log(boy.name);  // 'Keith'
console.log(boy.height);  // 180

const girl = new Person('Samsara', 160);
console.log(girl.name);  // 'Samsara'
console.log(girl.height); // 160
```
用以上的一个例子，来对构造函数的特点和new基本原理进行一个梳理。

上面代码中，首先，我们创建了一个构造函数Person，传入了两个参数name和height。构造函数Person内部使用了this关键字来指向将要生成的对象实例。

然后，我们使用new命令来创建两个对象实例boy和girl。

当我们使用new来调用构造函数时，new命令会创建一个空对象boy，作为将要返回实例对象。接着，这个控对象的原型会指向构造函数Person的prototype属性。也就是boy._proto_ === Person.prototype的。要注意的是空对象指向构造函数Person的prototype属性，而不是指向构造函数本身。然后，我们将这个空对象赋值给构造函数内部的this关键字。也就是说，让构造函数内部的this关键字指向一个对象实例。最后，开始执行构造函数内部代码。

因为对象实例boy和girl是没有name和height属性的，所以对象实例中两个属性都是继承自构造函数Person中的。这也就说明了构造函数是生成对象的函数，是给对象提供模板的函数。

一个问题，如果我们忘记使用new命令来调用构造函数，直接调用了构造函数，会发生什么？

这种情况下，构造函数就变成了普通函数，并不会生成实例对象。而且由于后面会说到的原因，`this`这时代表全局对象，将造成一些意想不到的结果。
```
function Keith() {
  this.height = 180;
}

const person = Keith();

console.log(person.height); // TypeError: person is undefined
console.log(window.height); // 180，在浏览器环境中执行时，window就代表全局对象
```
上面代码中，当在调用构造函数Keith时，忘记加上new命令。结果是this指向了全局作用域，height也就变成了全局变量。而变量person变成了undefined。

因此，应该非常小心，避免出现不实用`new`命令直接调用构造函数的情况。

为了保证构造函数必须与`new`命令一起使用一个解决办法是，在构造函数内部使用严格模式，即第一行加上`use strict`。
```
function Person(name, height) {
  'use strict';
  this.name = name;
  this.height = height;
}

const boy = Person();
console.log(boy)  // TypeError: name is undefined
```
上面代码的`Person`为构造函数，`use strict`命令保证了该函数在严格模式下运行。由于在严格模式中，函数内部的`this`不能指向全局对象。如果指向了全局，this默认等于`undefined`，导致不加`new`调用会报错（JavaScript不允许对`undefined`添加属性）。

另一个解决办法，是在构造函数内部判断是否使用`new`命令，如果发现没有，则直接返回一个实例对象。
```
function Person(name, height) {
  if (!(this instanceof Person)) {
    return new Person(name, height);
  }
  this.name = name;
  this.height = height;
}
const boy = Person('Keith');
console.log(boy.name)  // 'Keith'
```
上面代码中的构造函数，不管加不加`new`命令，都会得到同样的结果。

如果构造函数内部又`return`语句，而且`return`后面跟着一个复杂数据结构（对象，数组等），`new`命令会返回`return`语句指定的对象；如果`return`语句后面跟着一个简单数据类型（字符串，布尔值，数值等），则会忽略`return`语句，返回this对象。
```
function Keith() {
  this.height = 180;
  return {
    height: 200
  };
}

const boy = new Keith();
console.log(boy.height);  // 200

function Keith() {
  this.height = 100;
  return 200;
}

const boy = new Keith();
console.log(boy.height);   // 100
```
另一方面，如果对普通函数（内部没有`this`关键字的函数）使用new命令，则会返回一个空对象。
```
function Keith() {
  return 'this is a message';
}

const boy = new Keith();
console.log(boy)  // Keith {}
```
上面代码中，对普通的函数使用new命令，会创建一个空对象。这是因为new命令总是返回一个对象，要么是实例对象，要么是`return`语句指定的对象或数组。本例中`return`语句返回的是字符串，所以`new`命令就忽略了该语句。
