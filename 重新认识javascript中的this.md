`this`到底是个什么东西？在不同的执行条件之下`this`的指向总是让人琢磨不透。弄清楚`this`在不同环境下的指向，准确的辨认`this`的“真身”，是每一个javascript学习者都无法绕过的一个问题，也是一个非常重要的问题。平常代码中会遇到，而且几乎是面试的必考题。接下来我们就详细的探讨一下。在下面的介绍中如果没有特别说明都在在浏览器环境下。

在《javascript高级程序设计》（第三版）中是这样描述`this`的：
> `this`引用的是函数据以执行的环境对象——或者也可以说是`this`值。


## 执行环境
执行环境是一个比较复杂的对象，我们在这里可以简单的把它当成是**作用域链**。执行环境与作用域链实际上并不是相同的，但在这里并不影响我们对this的理解（执行环境是一个非常重要的概念，深入学习javascript请务必理解这个概念）。

执行环境可以分为两种：

- 全局执行环境
- 局部执行环境

##### 全局执行环境
全局执行环境就是代码运行最外部的一个执行环境。全局执行环境并不总是相同的，它会跟据宿主环境不同而发生变化，在浏览器中全局执行环境就是`window`对象，在node中就是global对象。全局执行环境是最先被创建的，当所有代码都执行完毕后才会被销毁，在执行栈的最底端。

##### 局部执行环境
局部执行环境就是函数执行环境。每个函数都有自己的执行环境，当执行流进入一个函数时，就会创建这个函数的执行环境，同时把这个执行环境推入到执行栈中。当这个函数执行完毕后，这个函数的执行环境就会被弹出执行栈并销毁。要注意的是函数每次被调用都会创建一个全新的执行环境，即使在函数内部调用自己也一样。

我们用一张图来表示执行环境与执行栈：![image](https://github.com/Hiker9527/js/blob/master/static/context.png)；

## this的指向
上面说过this是对函数执行环境的一个引用，确定this的指向就变成了确定当前函数的执行环境了。执行环境是在函数被调用的时候创建的，实际上this的指向也是在创建执行环境的过程中才被确认，并不是在函数声明时确定的，所以在调用之前我们无法确认this的指向。这就导致了同一个函数在不同情况下被调用时，它的this就会指向不同的执行环境对象。

#### 全局环境中的this
在浏览器环境中，全局环境中的this就是window对象，如果我们直接在最外层打印this，你就能看到它确实window对象：

window中的this；
![image](https://github.com/Hiker9527/js/blob/master/static/window_this.png);

从上面浏览器控制台的打印中可以验证this确实等于window对象的。全局环境中的this也是有权访问我们在全局定义的变量、函数的。

在node环境中，我们在一个js文件的最外层直接打印this，这个this是否也指向global对象。

```
console.log(this);         // {}, 输出一个空对象
console.log(global);       // {...}， 输出node的global对象

this.name = 'shawnWang';
console.log(this.name);    // 'shawnWang' 
console.log(global.name);  // undefined
```
为什么这里的`this`是一个空对象，而不是表现的像浏览器中一样指向`global`对象？这里其实跟Node的模块机制有关。node中每个文件都是一个模块，我们在当前文件中的代码并不是在真正的全局环境中执行，这些代码在运行前都会经过一次包装，所以这些代码最后都是在经过包装的模块中执行。当前代码在运行时，最外层的`this`指向的是`module.exports`，具体还要去了解node的模块机制。
```
this.a = 1;
console.log(module.exports.a);        // 1
console.log(this === module.exports); // true
```
如果在写代码过程中有定义全局变量的需求，需要显式的在`global`对象上定义（或者直接去掉定义变量前的关键字也可以，但是这并不是一个好的编码习惯，不推荐着么做）。

#### 方法调用中的this
当一个函数作为一个对象的属性时，我们就称这个函数是该对象的一个方法，执行这个对象的属性就是调用这个方法。方法调用中的`this`指向的就是它所属的对象。
```
this.name = 'Rofay';

const obj = {
    name: 'shawnWang',
    sayName: function() {
        console.log(this.name)
    }
}

obj.sayName();    // 'shawnWang'
```
这种情形是最普通也是比较容易理解的一种情形。

#### 函数调用中的this
按照上面this的解释，普通函数定义中的this应该指向这个函数外部的执行环境，但事实是所有的函数调用（匿名函数、声明函数）中，函数内部的`this`都指向了全局执行环境。
```
var name = 'Window Object';
// 声明一个函数
function sayWindowName() {
    console.log(this.name)
}

// 定义一个对象
const obj = {
    name: 'shawnWang',
    sayName: function() {
        // 在方法内部调用普通函数
        function anotherSayName() {
            console.log(this.name);
        }
        anotherSayName()
    }
}
// 函数调用
sayWindowName();  // 'Window Object'
// 方法调用
obj.sayName();    // 'Window Object'
```
上面的代码中，不管我们是在全局环境中执行`sayWindowName`还是在方法调用中执行`sayWindowName`，`sayWindowName`内部的`this`都指向了全局执行环境对象，也就是`window`对象。

`this`的值并不总是指向调用对象，在一些特殊的情况下，`this`的值可能会发生改变：
```
var name = 'Window Object';
const obj = {
    name: 'shawnWang',
    sayName: function() {
        console.log(this.name);
    }
}

obj.sayName();                  // 'shawnWang'
(obj.sayName)();                // 'shawnWang'
(obj.sayName = obj.sayName)();  // 'Window Object';
```
前两次调用比较好理解，就是简单的方法调用，第三次函数在调用之前先执行了一个赋值语句，然后再调用赋值后的结果，赋值的结果就是这个函数本身，所以最后就是相当于调用了一个函数，这里的`this`就指向了全局执行环境。

函数调用的全局性不知道是javascript语言设计者故意为之还是一个BUG，但是这确实在我们的使用中带来了一些麻烦。如果确实要用到外层执行环境对象，那么就在外层执行环境中将`this`赋值给另外一个变量，通常会把这个变量命名为`that`。
```
const obj = {
    name: 'shawnWang',
    sayName: function() {
        const that = this;
        function anotherSayName() {
            console.log(that.name);
        }
        anotherSayName()
    }
}
```
#### 闭包中的this
在闭包中使用`this`一定要非常小心，闭包中的`this`是最难以捉摸的一种情形。全局环境的this指向全局执行环境，方法调用中的this指向调用方法的对象，函数调用中的this指向全局执行函数，下面看一个关于闭包的例子：
```
var name = 'Window Object';

const obj = {
    name: 'shawnWang',
    sayName: function() {
        return function() {
            console.log(this.name);
        }
    }
}

obj.sayName()();   // 'Window Object'

```
调用`obj.sayName`后返回一个匿名函数，然后紧接着就调用这个匿名函数，此时这个匿名函数内部的`this`就指向了全局执行环境。如果要在闭包中访问外层环境中的`this`也可以采用我们上面提到的将它赋值给`that`变量来实现。

#### 构造函数中的this

构造函数中this比较简单。用`new`命令调用构造函数时，构造函数内部会自动创建一个空的对象，同时`this`会被绑定到这个新的对象上。构造函数内部的`this`也可以理解为这个构造函数实例化后的那个实例对象。

#### apply、call调用中的this

在javascript中函数也是对象，所以函数也可以拥有属性和方法。函数有两个可以绑定`this`值的两个方法`call`和`apply`。其实这两个方法并没有本质的区别，都是用来为函数绑定`this`值的，具体的区别就不在这里讨论了。下面我们以`apply`为例，看看怎么样用它们绑定`this`值。

```
const obj1 = {
    name: 'Rofay'
}

const obj2 = {
    name: 'shawnWang',
    sayName: function() {
        console.log(this.name);
    }
}
obj2.sayName();               // 'shawnWang'
obj2.sayName.apply(obj1);     // 'Royfay'

```
用普通的方式执行`obj2.sayName`时，方法内部的`this`指向调用它的对象`obj2`，所以`this.name`就是`'shawnWang'`。当我们用`apply`将`obj1`绑定到`obj2.sayName`方法的`this`值时，此时内部的`this`会指向`obj1`，所以我们就看到会打印出`'Rofay'`。

`apply`方法还可以接收第二个数组参数作为被调用方法的参数。

除了这两个方法外，在ES5中又新增了一个`bind`的方法，也是用来绑定函数作用域的，不同的是`bind`方法会返回一个新的函数，在执行这个新函数时，会将这个函数内部的`this`值指向之前传入的要绑定的对象。
```
const obj1 = {
    name: 'Rofay'
}

const obj2 = {
    name: 'shawnWang',
    sayName: function() {
        console.log(this.name);
    }
}

obj2.sayName.bind(obj1)();    // 'Rofay'

```
这个方法主要用在事件处理程序以及`setTimeout`和`setInterval`中。但是`bind`方法也会带来额外的内存开销，所以除了有必要，不要滥用`bind`方法。

#### 箭头函数中方的this
ES6中给我们提供了一种新的定义函数的形式，除了更加简洁之外，还有一个让人兴奋的特性。前面提到函数中`this`的指向是在函数执行的时候确定，而并不是在声明的时候确定，但是在箭头函数体内`this`的指向就是在声明时所在的执行环境。这样就消除了函数因为不同的执行环境而导致的`this`指向的不确定性。
```
var name = 'Rofay';

const obj = {
    name: 'shawnWang',
    sayName: function() {
        const anotherSayName = () => {
            console.log(this.name);
        }
        // const anotherSayName = function() {
        //     console.log(this.name)
        // }
        setTimeout(anotherSayName, 100)
    }
}

obj.sayName(); // 'shawnWang'
```
这里控制台会打印出`obj.name`的值，而不是`window.name`的值。这是因为`sayName`方法内部定义`anotherSayName`函数时我们使用的时箭头函数的形式，在定义这个函数的时候，它的`this`就已经被绑定到了它所在的执行环境中，所以在`setTimeout`中调用时它的`this`并没有指向全局执行环境。

## 总结

上面的概括基本上可以应付在javascript大多数的`this`场景了。编写代码中`this`不可避免，不要害怕使用this，如果确实不清楚在当前函数中this的指向，最好的办法就是打印出来，看看此时的`this`到底指向的哪个作用域。


