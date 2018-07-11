原文来自Ben Alman的Partial [Application in JavaScript](http://benalman.com/news/2012/09/partial-application-in-javascript/)

如果你没有用过用过其它的编程语言比如ML或者Haskell，JavaScript中的[partial application](https://en.wikipedia.org/wiki/Partial_application)和[currying](https://en.wikipedia.org/wiki/Currying)这些概念对你来说可能有些陌生。这真是令人难过，一旦你理解你这些概念，你就能把它们用在你自己的代码中。

注意：这篇文章的原始出处在[MSDN Script Junkie](https://msdn.microsoft.com/en-us/magazine/gg575560.aspx)的网站，不过它已经被重写了。那一个版本更加直观，不过这这版本更接近当前也更准确。

### 函数（Functions）
即使你已经知道JavaScript函数可以返回一个函数，并且也能接收一个函数作为参数，不过我依然建议你读完第一节当作一次复习。如果你已经知道这个东西，你可以跳过这一节直接到Partial Application这一节。

让我们先看一个非常基础的例子：
```
function add(a, b) {
    return a + b;
}

add(1, 2);      // 3
add(1, 3);      // 4
add(1, 10);     // 11
add(1, 9000);   // 9001
```
这个例子非常简单，想象一下在这个场景中你必须重复的调用一个函数，而且每次传入的第一个参数都相同。不必要的重复是导致错误的一个主要因素，你能解决这个问题而且不需要重复这段代码的一个办法就是就是把需要重复的值保存在一个变量中，然后在你每次需要需要这个值的时候就可以在这个变量中获得。
```
function add(a, b) {
    return a + b;
}
var value = 1;

add(value, 2);        // 3
add(value, 3);        // 4
add(value, 10);       // 11
add(value, 9000);     // 9001
```
如你所见，用一个变量代替参数明显地让这段代码变的更加容更新和维护了。另一方面，这段代码还是没有避免重复。如果同样的值将会被加多次，这也许对我们创建一个拥有这个内置行为的专门函数有些帮助。

### 函数引用函数（Functions Invoking Functions）
不管你是为你自己写代码还是给你的用户提供一个API，如果你希望一个函数会被用相同的参数重复调用，通常来说给一个更通用的函数外围创建一个更加专门的“wrapper”函数是很有用。

要达到这个目的，有一种方法就是除了定义通用函数外，还要手动定义一个特有函数。这在JavaScript中很容易，因为函数可以引用另一个函数。
```
// More general function.
function add(a, b) {
    return a + b;
}

add(1, 2);   // 3
add(10, 3);  // 13

// More specific functions.
function addOne(b) {
    return add(1, b);
}

addOne(2);    // 3
addOne(3);    // 4

function addTen(b) {
    return add(10, b);
}

addTen(2);    // 12
addTen(3);    // 13
```
用这种方式定义少量的特有函数是非常简单直接的，如果你有足够多的这样的函数就会产生大量的额外代码，还的维护。

### 函数返回函数（Functions Returning Functions）
这里你会看到创建一个`makeAdder`函数的可能，当用一个参数调用这个`makeAdder`函数就会返回一个新的函数（能生成另外一个函数或者对象的函数通常被叫做工厂）。被返回的函数，当被用一个参数调用时，把参数的值和开始指定（或者绑定）的参数的值加在一起，返回求和结果。
```
// More general function.
function add(a, b) {
    return a + b;
}

add(1, 2);   // 3
add(10, 3);  // 13

// More specific function generator
function makeAdder(a) {
    return function(b) {
        return a + b;
    }
}

// More specific functions.
var addOne = makeAdder(1);
addOne(2);    // 3
addOne(3);    // 4

var addTen = makeAdder(10);
addTen(2);    // 12
addTen(3);    // 13
```
这能成为可能是因为JavaScript支持闭包，它允许函数访问被调时的包含环境作用域里的变量。除此之外，在JavaScript中函数是一等公民。因此函数既可以作为参数也可以被返回。闭包和一等公民通常会一起使用，让返回的函数可以继续保持对传入参数的访问能力。

这个例子给你提供了用`addOne(2)`代替`add(1, 2)`的便利，但是它也会带来一些开销。首先，真实的加法逻辑在普通函数`add`和工厂函数`makeAdder`中产生了重复，如果在不重复的代码中就会造成一些困难。其次，你需要手动为想要在这种方式中管理的每一个不同的`something`函数创建一个`makeSomething`函数。
### 函数接收函数（Functions Accepting Functions）
下一步就是创建一个更抽象的工厂函数，这个函数不仅可以接收一个用来"绑定"的参数，也可以接收一个包含所有核心逻辑的函数用来被调用（函数被当作参数传入另一个函数通常被称为回调）。

我们会创建一个简单的工厂函数，它可以被用来创建任意函数的绑定版本（bound versions）。

需要注意的是不能用任何的方式修改源函数，它们的行为也就不会发生改变。它们只是被"wrapper"函数简单的引用和调用。
```
//  Relatively flexible, more specific function generator.
function bindFirstArg(fn, a) {
    return function(b) {
        return fn(a, b)
    }
}

// More general functions.
function add(a, b) {
    return a + b;
}

add(1, 2);           // 3

function multiply(a, b) {
    return a * b;
}

multiply(2, 10)      // 20

// More specific functions.
var addOne = bindFirstArg(add, 1);
addOne(2);           // 3
addOne(3);           // 4
addOne(10);          // 11
addOne(9000);        // 9001

var multiplyByTen = bindFirstArg(multiply, 10);
multiplyByTen(2);    // 20
multiplyByTen(3);    // 30
multiplyByTen(10);   // 100
multiplyByTen(9000); // 90000
```
特别有趣的是`bindFirstArg`函数不仅可以用来给任意函数绑定第一参数，也可以用来给自己绑定一个函数作为第一参数，如此我们就创建了一个“可绑定的(bindable)”函数。

可以这样想：如果`bindFirstArg`函数可以给接收两个参数的函数绑定第一个参数——就像给`add`绑定`1`或者是给`multiply`绑定`2`，`bindFirstArg`函数也接收两个参数，`bindFirstArg`可以将一个函数作为第一参数绑定给自己就是合理的。

现在我们创建了一个更具体的`bindFirstArg`版本。
```
  var makeAdder = bindFirstArg(bindFirstArg, add);

  // More specific functions.
  var addOne = makeAdder(1);
  addOne(2); // 3
  addOne(3); // 4

  var addTen = makeAdder(10);
  addTen(2); // 12
  addTen(3); // 13
```
是不是看着很熟悉？的确。这就是`makeAdder`函数，只不过是用了一个比之前更加通用的方式创建的。

现在`bindFirstArg`函数比之前的例子更灵活了，不过也仅仅是灵活了一点点而已如果你想绑定的不只是一个参数呢？如果你有一个函数接收三个或者更多个参数，你可能想根据情况绑定第一个参数或者是前两个或者任意数量的参数呢？

尽管这个解决方案比之前的更灵活，它也会更通用。

### 局部应用（Partial Application）
[局部应用](https://en.wikipedia.org/wiki/Partial_application)可以这样来描述：一个接收一定数量的参数的函数，给这些参数中的的一个或多个绑定上值，并且返回一个只接收剩余未被绑定上值的参数。

这句话的意思就是，给定一个任意函数，就会生成一个已经绑定了一个或多个参数的新函数，或者部分应用。如果你仔细看的话你现在就应该明白之前的例子实际上已经示范了部分应用，尽管还有一些限制。

如果你用过ECMAScript5的Function#bind方法，这个方法允许函数给自己绑定`this`值和一些参数，其实你已经对部分应用很熟悉了。Although with Function#bind, it might help to think of the this value as an implicit 0th argument--see the Extra Credit section at the end for a few Function#bind examples.

#### 部分应用：从左边开始
这个例子的灵活性要比前面的有明显的提高，因为它使用了arguments对象来动态的决定要绑定的参数的数量。

需要注意的是`arguments`对象是函数执行时创建的一个类数组对象，包含了传入这个函数中的所有参数，而且只能在这个函数中才能访问得到。`arguments`只是一个类数组对象，它并不是一个真正的数组。这意味这它拥有`.length`属性和数字类型的索引值，但是它没有任何的数组方式，比如`.concat`或者`.slice`。为了吧`arguments`对象转换成一个数组，就需要使用call调用方式来调用Array#slice方法。

下面的`partial`函数返回一个匿名函数，当执行这个匿名函数时，就会用先前绑定的参数和后来传递给匿名函数的所有参数来调用`fn`函数。
```
function partial(fn, /*, args... */) {
    // A reference to tha Arrar#slice method.
    var slice = Array.prototype.slice;
    // Convert arguments object to an array, removing the first argument
    var args = slice.call(arguments, 1);
    
    return function() {
        // Invoke the originally-specified function, passing in all 
        // originally-specified arguments, followed by any 
        // just-specified arguments.
        fn.apply(this, args.concat(slice.call(arguments, 0)));
    };
}
```
这里有一个使用了`partial`函数的部分应用的例子：
```
// Add all arguments pass in by iterating over the `arguments` object.
function addAllTheThings() {
    var sum = 0;
    for (var i = 0; i < arguments.length; i++) {
        sum += arguments[i];
    }
    return sum;
}

addAllTheThings(1, 2);                 // 3
addAllTheThings(1, 2， 3);             // 6
addAllTheThings(1, 4, 9, 16, 25);      // 55

// More specific functions.
var addOne = partial(addAllTheThings, 1);
addOne();                              // 1
addOne(2);                             // 3
addOne(2, 3)                           // 6
addOne(4, 9, 16, 25);                  // 55

var addTen = partial(addAllThings, 1, 2, 3, 4);
addTen();                              // 10
addTen(2);                             // 12
addTen(2, 3);                          // 15
addTen(4, 9, 16, 25);                  // 64
```
当`partial`函数执行的时候，原始传进去的参数在去掉第一个`fn`参数后会被存储到`args`数组中。每次返回的匿名函数被调用，就会用apply调用方式执行开始被传进去的`fn`函数。`.apply()`接收一个用由两个数组通过`.concat`结合的数组，这样就能用当前传入的和先前绑定的参数来调用`fn`函数了。

#### 完全应用（Full application）?
值得注意的是部分应用只有在部分使用一个函数的参数的情况下才非常有用。如果你在开始就将所有的参数都传进去，那么将来这个函数的行为跟硬编码没什么区别。
```
function add(a, b) {
    return a + b;
}

var alwaysNine = partial(add, 4, 5);
alwaysNine();         // 9
alwaysNine(1);        // 9 - this is just calling add(4, 5, 1)
alwaysNine(9001);     // 9 - this is just calling add(4, 5, 9001)
```
在JavaScript中，如果传入的参数比函数期望多的话，函数就会忽略多出的部分（除非是用`arguments`对象访问的）。所以无论给`alwaysNine`函数传入多少参数，它的结果都不会发生改变。

#### 部分应用：从右边开始（Partial Application: From the Right）
在这之前的所有部分应用的例子展示的都是绑定*leftmost*函数参数这一种情况。这也是部分应用最常用的情况，但不是唯一的一种。

用与`partial`函数相似的代码可以很容易创建一个绑定*rightmost*函数参数的`partialRight`函数。事实上，所有需要改变的就只是用当前(just-specified)参数的来结合原始(originally-specified)参数。

下面的`partialRight`函数返回一个匿名函数，当这个匿名函数被调用时就会用传入匿名函数的参数加上原先绑定的参数调用`fn`函数。
```
function partialRight(fn, /*, args... */) {
    // A reference to tha Arrar#slice method.
    var slice = Array.prototype.slice;
    // Convert arguments object to an array, removing the first argument
    var args = slice.call(arguments, 1);
    
    return function() {
        // Invoke the originally-specified function, passing in all 
        // just-specified arguments, followed by any 
        // originally-specified arguments.
        fn.apply(this, slice.call(arguments, 0).concat(args));
    };
}
```
下面是部分应用的例子，看看`parlial`和`partialRight`的差别：
```
function wedgie(a, b) {
  return a + ' gives ' + b + ' a wedgie.';
}

var joeGivesWedgie = partial(wedgie, 'Joe');
joeGivesWedgie('Ron');    // "Joe gives Ron a wedgie."
joeGivesWedgie('Bob');    // "Joe gives Bob a wedgie."

var joeReceivesWedgie = partialRight(wedgie, 'Joe');
joeReceivesWedgie('Ron'); // "Ron gives Joe a wedgie."
joeReceivesWedgie('Bob'); // "Bob gives Joe a wedgie."
```
`partialRight`执行过程中唯一的一个问题就是，如果我们给部分应用的函数传入了过多的参数的话，那么之前绑定的参数就会被取代，这样之前的绑定函数就失去了作用。
```
joeReceivesWedgie('Bob', 'Fred'); // "Bob gives Fred a wedgie."
```
更健壮的做法应该是还要将函数参数的数量考虑在内，但是这样会增加函数的复杂性。

*在JavaScript中，从左边的部分应用比从右边开始的更简单更健壮*。

#### 部分应用：从任意位置开始（Partial Application: From anywhere）
`partial`和`partialRight`函数不管是可以从左边还是右边实现部分应用，都不能阻止你更进一步的创建一个可以指定参数去部分应用的函数。wu.js和函数式JavaScript库都又一个叫做`partial`的方法，这个方法可以通过提供占位符来实现这种部分应用方式。在下面的案例中，我会给这个这个函数取名为`partialAny`，这个函数的占位符值是一个任意的属性，取名为`partialAny._`。

下面的`partialAny`函数返回一个匿名函数，当这个匿名函数被调用时，它会用原始绑定的参数调用`fn`函数。所有的“占位符”原始参数都应该用传递给匿名函数的参数按顺序替换。所有传给匿名函数剩余的参数都将被追加到末尾。

要注意的是如果对你`(function(){ /* code */ }())`这种模式不熟悉，请阅读我的[ article on IIFEs](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)aka。立即执行函数表达式。
```
var partialAny = (function() {
  // A reference to the Array#slice method.
  var slice = Array.prototype.slice;
  // This function will be returned as a result of the IIFE and assigned
  // to the external `partialAny` var.
  function partialAny(fn /*, args...*/) {
  // Convert arguments object to an array, removing the first argument.
    var orig = slice.call(arguments, 1);
    
    return function() {
      // Convert arguments object to an array.
      var partial = slice.call(arguments, 0);
      var args = [];
      
      // Iterate over the originally-specified arguments. If the argument
      // was the `partialAny._` placeholder, use the next just-passed-in
      // argument, otherwise use the originally-specified argument.
      for (var i = 0; i < orig.length; i++) {
        args[i] = orig[i] === partialAny._ ? partial.shift() : orig[i];
      }
      
      // Invoke the originally-specified function, passing in interleaved
      // originally- and just-specified arguments, followed by any remaining
      // just-specified arguments.
      return fn.apply(this, args.concat(partial));
    }
  }
  // This is used as the placeholder argument.
  partialAny._ = {};
  return partialAny;
}());
```
这里有一个使用`partialAny`的例子。

请注意，示例中了用一个叫做`__`的变量来代替繁琐的`partialAny._`，让代码看起来更友好。当然也可以用`foo`或者`PLACEHOLDER`替代`__`。
```
function hex(r, g, b) {
    return '#' + 'r' + 'g' + 'b';
}

hex('11', '22', '33');      // '#112233'

// A more visually-appealing placeholder.
var __ = partialAny._;

redMax = partialAny(hex, 'ff', __, __);
redMax('11', '22');        // '#ff1122'

greenMax = partialAny(hex, __, 'ff');
greenMax('33', '44');        // '#ff3344'

blueMax = partialAny(hex, __, __, 'ff');
blueMax('55', '66');        // '#ff5566'

magentaMax = partialAny(hex, 'ff', __, 'ff');
magentaMax('77');        // '#ff77ff'
```
一些库把`partialAny`的功能暴露为`partial`，它们并没有定义另外一个叫做`partial`的函数，它们把从左边的部分应用函数叫做`curry`。

把部分应用和柯里化联系在一起是一个可悲的但是又非常常见的问题，实际上它们是两个不同的概念。

Note that the remainder of this article describes currying, which is somewhat academic and has limited practical use in JavaScript. While you're encouraged to continue reading, if your brain is on fire, you may want to skip to the article's Final Words. That being said, if you do skip the the end, you'll miss the crazy stuff.

Addendum: there's now even more crazy stuff at the very end, in the Extra Credit section. Good luck.

（译者注：大概意思就是文章下面的currying部分有些晦涩难懂，如果你实在是不想继续看下去就会错过很重要的东西。还有更重要的东西在Extra Credit这一节中）。

### 柯里化（Currying）
柯里化可以被描述为：将一个具有多个参数的函数转化为可以链式调用多个单参函数。

这就意味着，一旦一个函数被柯里化了，那它首先就要部分应用，因为只要你给一个柯里化函数传入一个参数，你就部分应用了这个参数。不想部分应用，一个柯里化的函数会持续返回柯里化函数直到所有的参数都被赋值。

下面的`curry`函数返回一个匿名函数f，这个匿名函数期望接收一个参数。当这个匿名函数被调用时，它会检查是否`fn`期望的参数都被赋值了。如果条件成立，就用这些参数调用`fn`。否则就返回另一个与匿名函数f有相同行为的匿名函数f1。用递归来维持一个包含已传入参数的数组。一旦`fn`所有的参数都被传了进来，`fn`就会被调用。

请注意，JavaScript函数有一个`.length`属性来反应函数的参数个数，在某些情况下JavaScript无法确定期望的参数的数量（比如函数在内部用`argumensts`对象代替独立的参数）。

在那些函数的数量不能自动确定的情况中，你可以指定一个数值类型的参数`n`来代替`fn.length`属性。
```
function curry(fn, n) {
  // If `n` argument was omitted, use the function .length property.
  if (typeof n !== 'number') {
    n = fn.length;
  }

  function getCurriedFn(prev) {
    return function(arg) {
      // Concat the just-specified argument with the array of
      // previously-specified arguments.
      var args = prev.concat(arg);
      if (args.length < n) {
        // Not all arguments have been satisfied yet, so return a curried
        // version of the original function.
        return getCurriedFn(args);
      } else {
        // Otherwise, invoke the original function with the arguments and
        // return its value.
        return fn.apply(this, args);
      }
    };
  }

  // Return a curried version of the original function.
  return getCurriedFn([]);
}
```
这里有一个完整的柯里化的例子，使用了`curry`函数：
```
var i = 0;
function a(arg1, arg2, arg3) {
  return ++i + ': ' + arg1 + ', ' + arg2 + ', ' + arg3;
}

// Normal function invocation.

a('x', 'y', 'z'); // "1: x, y, z"
a('x', 'y');      // "2: x, y, undefined"
a('x');           // "3: x, undefined, undefined"
a();              // "4: undefined, undefined, undefined"

// Curried function invocation.

var b = curry(a);
b();              // `a` not invoked, curried function returned
b('x');           // `a` not invoked, curried function returned
b('x')('y');      // `a` not invoked, curried function returned
b('x')('y')('z'); // "5: x, y, z"
b('x')('y')();    // "6: x, y, undefined"
b('x')()();       // "7: x, undefined, undefined"
b()('y')();       // "8: undefined, y, undefined"
b()()('z');       // "9: undefined, undefined, z"
b()()();          // "10: undefined, undefined, undefined"

var c = b('x');
c();              // `a` not invoked, curried function returned
c('y');           // `a` not invoked, curried function returned
c('y')('z');      // "11: x, y, z"
c('y')();         // "12: x, y, undefined"
c()('z');         // "13: x, undefined, z"
c()();            // "14: x, undefined, undefined"

var d = c('y');
d('z');           // "15: x, y, z"
d();              // "16: x, y, undefined"

var e = d('z');
e;                // "17: x, y, z"
```
#### 手动指定函数参数数量（Manually Specifying Function Arity）
下面的例子中，当柯里化`a1`时你需要给`n`赋一个值，因为`a1.length`是`0`。这是因为在`a1`定义的时候没有指定参数列表，JavaScript无法确定函数内部到底会使用多少个参数。
```
var i = 0;
function a1() {
  var arg1 = arguments[0];
  var arg2 = arguments[1];
  var arg3 = arguments[2];
  return ++i + ': ' + arg1 + ', ' + arg2 + ', ' + arg3;
}

// Normal function invocation.

a1('x', 'y', 'z'); // "1: x, y, z"
a1('x', 'y');      // "2: x, y, undefined"
a1('x');           // "3: x, undefined, undefined"
a1();              // "4: undefined, undefined, undefined"

// Curried function invocation.

var b1 = curry(a1, 3);
b1();              // `a` not invoked, curried function returned
b1('x');           // `a` not invoked, curried function returned
b1('x')('y');      // `a` not invoked, curried function returned
b1('x')('y')('z'); // "5: x, y, z"
b1('x')('y')();    // "6: x, y, undefined"
b1('x')()();       // "7: x, undefined, undefined"
b1()('y')();       // "8: undefined, y, undefined"
b1()()('z');       // "9: undefined, undefined, z"
b1()()();          // "10: undefined, undefined, undefined"

var c1 = b1('x');
c1();              // `a` not invoked, curried function returned
c1('y');           // `a` not invoked, curried function returned
c1('y')('z');      // "11: x, y, z"
c1('y')();         // "12: x, y, undefined"
c1()('z');         // "13: x, undefined, z"
c1()();            // "14: x, undefined, undefined"
```
如果给`n`赋了一个不同的值，那么被柯里化的函数就只期望那么多数量的参数，忽略原始函数的参数个数。

#### 部分应用 vs 柯里化（Partial Application vs Currying）
那么部分应用与柯里化的区别在哪里？对比一下如何部分应用和柯里化函数的行为：
```
function add(a, b, c) {
    var sum = a + b + c;
    return a + ' + ' + b + ' + ' + c + ' = ' + sum;
}

add(1, 2, 3);    // '1 + 2 + 3 = 6'

// Partial application.

var addOnePartial = partial(add, 1);
addOnePartial(2, 3);     // "1 + 2 + 3 = 6"
addOnePartial(2);        // "1 + 2 + undefined = NaN"

// Currying.

var addOneCurried = curry(add)(1);
addOneCurried(2)(3); // "1 + 2 + 3 = 6"
addOneCurried(2);    // `add` not invoked, curried function returned
```
一个部分应用的函数不关心给它传递了多少个参数；当被调用的时候，它会合并当前传进来的参数和原始参数，来调用原始函数。

一个柯里化了的函数，实际上是一个有多个函数的链，每个函数都只接收单个参数；只有当链上所有的函数被调用，才会用所有赋值的参数来调用原始函数。

#### 部分应用柯里化：弗兰肯斯坦的怪物（Partial Applicurrying: Frankenstein's Monster）
柯里化从概念上来说是比较难以理解的。就算是认真研究过还是经常会把柯里化和部分应用混淆在一起，我在这篇文章的原始版本中就弄错了。我写的`curry`函数实际上是一个介于部分应用和柯里化的奇怪的杂交体。真是让人难过，我想简单的给你们展示一下我曾见创建的这个恶心的东西。

这个例子跟前面的`curry`例子非常相似，但是有一个关键的不同点是：用`arguments`对象中动态解析出来的参数代替每个柯里化的函数要求的单个参数（见下一段加粗文字）。

所以，下面的`frankenCurry`函数返回一个接收**零个或多个参数**的匿名函数*f*。当被调用时，它会检查`fn`函数期望的参数是不是都已经得到了满足。如果是的话，就用这些参数调用`fn`函数。否则，就返回另一个与匿名函数*f*行为相同的匿名函数*f1*。用递归来维持一个保存了已传入参数的数组。一旦`fn`期望的所有参数被满足，`fn`就会被执行。

柯里化函数是一个有N（译者注：参数的个数）个函数的链，`frankenCurry`是一个想要多少就可以是多少个函数的链，直到所有的参数被满足。
```
function frankenCurry(fn, n) {
  // If `n` argument was omitted, use the function .length property.
  if (typeof n !== 'number') {
    n = fn.length;
  }

  function getFrankenCurriedFn(prev) {
    return function() {
      // Concat all just-specified arguments with the array of
      // previously-specified arguments. Madness!
      // 译者注：只有这一行与 curry 函数不同
      var args = prev.concat(Array.prototype.slice.call(arguments, 0));
      if (args.length < n) {
        // Not all arguments have been satisfied yet, so return a
        // franken-curried version of the original function.
        return getFrankenCurriedFn(args);
      } else {
        // Otherwise, invoke the original function with the arguments and
        // return its value.
        return fn.apply(this, args);
      }
    };
  }

  // Return a franken-curried version of the original function.
  return getFrankenCurriedFn([]);
}
```
如果你总是给每个 franken-curried 函数都传入一个参数，那么它的行为与`frankenCurry`函数完全一致，但是如果你给 franken-curried函数传入的不是一个参数的时，那么它的行为就完全不同了。
```
var i = 0;
function a(arg1, arg2, arg3) {
  return ++i + ': ' + arg1 + ', ' + arg2 + ', ' + arg3;
}

// Curried function invocation.

var b = curry(a);
b();              // `a` not invoked, curried function returned
b('x');           // `a` not invoked, curried function returned
b('x')('y');      // `a` not invoked, curried function returned
b('x')('y')('z'); // "1: x, y, z"
b('x')('y')();    // "2: x, y, undefined"
b('x')()();       // "3: x, undefined, undefined"
b()('y')();       // "4: undefined, y, undefined"
b()()('z');       // "5: undefined, undefined, z"
b()()();          // "6: undefined, undefined, undefined"

// Franken-curried function invocation.

var c = frankenCurry(a);
c();              // `a` not invoked, curried function returned
c('x');           // `a` not invoked, curried function returned
c('x')('y');      // `a` not invoked, curried function returned
c('x')('y')('z'); // "7: x, y, z"
c('x')('y')();    // `a` not invoked, curried function returned
c('x')()();       // `a` not invoked, curried function returned
c()('y')();       // `a` not invoked, curried function returned
c()()('z');       // `a` not invoked, curried function returned
c()()();          // `a` not invoked, curried function returned
c('x')('y', 'z'); // "8: x, y, z"
c('x', 'y')('z'); // "9: x, y, z"
c('x', 'y', 'z'); // "10: x, y, z"
```
如果你读过[ IIFE 这篇文章](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)，你就会知道函数表达式的调用并不需要赋值给一个命名变量；你要做的全部就是在函数表达式后面放置一对括号。因为franken-curried 函数会持续返回一个函数直到所有的参数被满足，你可以写一些看起来很疯狂但确实有效的JavaScript。
```
function add(a, b, c) {
  var sum = a + b + c;
  return a + ' + ' + b + ' + ' + c + ' = ' + sum;
}

frankenCurry(add)(1)(2)(3);             // "1 + 2 + 3 = 6"
frankenCurry(add)(1, 2)(3);             // "1 + 2 + 3 = 6"
frankenCurry(add)(1, 2, 3);             // "1 + 2 + 3 = 6"
frankenCurry(add)(1)(2, 3);             // "1 + 2 + 3 = 6"
frankenCurry(add)(1)()(2)()(3);         // "1 + 2 + 3 = 6"
frankenCurry(add)()(1)()()(2)()()()(3); // "1 + 2 + 3 = 6"
frankenCurry(add)()()()()()(1)()()()()()(2)()()()()()(3); // "1 + 2 + 3 = 6"
```
首先我承认 franken-curried 在现实环境中吸引力有限，不过换句话说，在JavaScrit中这就是柯里化。另外，我不会在真实环境中用这样的例子，我只是为了看起来很疯狂的JavaScript。

#### 柯里化：JavaScript中的应用（Currying: Practical Uses in JavaScript）
JavaScript中有实际使用柯里化的吗？不完全是。

在JavaScript中，柯里化显然很有必要，它没有像在其它的函数式编程语言中比如 ML和 Haskell中一样有用。这主要是由于JavaScript的函数行为与其它语言有些不同；JavaScriptz中的柯里化只是模拟柯里化而已。

电子书[Lear You a Haskell for Great Good](http://learnyouahaskell.com/)高阶函数这一章包含了这些差异。作者解释到：“每个Haskell官方的函数都只接收一个参数，所以我们如何才能定义和使用接收多个参数的函数？有一个很巧妙的技巧，所有接收多个参数的函数都被柯里化。”

实际上这些语言中，定义一个接收多个参数的函数其实就是定义一系列单参函数的语法糖，只是柯里化被内建到了语言的底层。这是因为在这些语言中，"function"的含义与在JavaScript中是不同的。它更像是 [数学中的函数](https://en.wikipedia.org/wiki/Function_(mathematics))，在这里你传入一个值得到一些输出，像是`f(x) = x + 1`。

因为这些语言中的每个函数都是柯里化后的，每个函数都可以简单的通过传入比它期望的少的参数，对它部分调用。如果你着么做的化就得到一个部分应用的函数。

在JavaScript中，函数的参数是可选的，(如果漏掉的化默认为`undefined`)，你不用特殊功能的函数比如`partial`或者`curry`的化就不能实现部分应用。这是因为用少于函数期望的参数个数来调用这个函数的话，这就跟给漏掉的参数传入`undefined`是一样的效果。

### 结语（Final Words）
部分应用最常见的用法就是在一个函数开始给它绑定一个或多个参数，比如上面的`partial`例子。最容易看到它的用法就是在Function#bind中，它不仅允许指定函数的上下文，用允许对它的参数从左边部分应用。

部分应用其它的变体也很有用，但不是很流行。这真是让人难过，它们绑定参数的能力允许你去部分应用一个接收参数的函数，并且让它更专业更容易使用。

虽然我在这里写了一下方便（也不是很方便）实用的函数，我还是推荐去看看有良好文档又流行的 wu.js和JavaScript函数式编程的库，比如 Lo-Dash 和 Underscore.js 库，它们支持了大多数。如果还不够你需要在JavaScript中利用部分应用实现需要的功能。

### 额外部分（Extra Credit）
之前在文章的前面提到，ECMAScript 5 Function#bind 方法就是一个部分应用，允许一个函数绑定`this`值和几个可选的参数。

这里我们有一个基础的例子来说明一个函数内部的`this`值依赖于函数的调用方式，和`.bind`方法是如何永久的锁定`this`值和可选的部分应用参数的。
```
var prop = 9001;
var obj = {
  prop: 1,
  add: function(a, b) {
    var sum = this.prop + a + b;
    return this.prop + ' + ' + a + ' + ' + b + ' = ' + sum;
  }
};

// When invoked this way, `this` inside `add` is `obj`
obj.add(2, 3);             // "1 + 2 + 3 = 6"

// When invoked like this, `this` inside `badAdd` is the global object.
var badAdd = obj.add;
badAdd(2, 3);              // "9001 + 2 + 3 = 9006"

// The `this` value can be explicitly set using call and apply invocation.
badAdd.call(obj, 4, 5);    // "1 + 4 + 5 = 10"
badAdd.apply(obj, [6, 7]); // "1 + 6 + 7 = 14"

// When a `this` value is bound, it stays bound.
var goodAdd = obj.add.bind(obj);
goodAdd(8, 9);             // "1 + 8 + 9 = 18"

// The same goes for partially applied arguments, too.
var goodAddEvenMore = obj.add.bind(obj, 10);
goodAddEvenMore(11);       // "1 + 10 + 11 = 22"

// Note that only a reference to the `this` object itself gets locked in,
// its properties can still be changed.
obj.prop = 99;
goodAdd(5, 6);             // "99 + 5 + 6 = 110"
goodAddEvenMore(7);        // "99 + 10 + 7 = 116"
```
现在那些基本的原理都被覆盖了...
##### 额外的，额外部分（Extra, Extra Credit）
在函数接收函数那一章中，`bindFirstArg`函数把自己传递给了自己，创建了一个“可绑定”函数，[Dave Herman](https://twitter.com/littlecalculist)在这三个方法中更进了一步，向大家展示了创建一个独立的`bind`函数是多么容易，将一个函数作为`call()`和`apply()`方法的第一个参数，代替必须将`call()`和`apply`作为这个函数的方法来调用。
```
var bind = Function.prototype.call.bind(Function.prototype.bind);
var call = bind(Function.prototype.call, Function.prototype.call);
var apply = bind(Function.prototype.call, Functionprototype.apply);
```
在这里我并不打算对这三行惊人的JavaScript代码多说什么，但是我在创建一个golfed-down 版本中实现了同样的功能。

这三行代码有点费脑，它说明了一个很有趣的点：在调用`.call`或者`.apply`是，你没有必要明确的给它们设置一个`this`值。

这到底是是什么意思呢？看一看下面的例子，这两个看起来完全不同的语句实际上做了同一件事：
```
fn.call(thisValue);
Function.prototype.call.call(fn, thisValue);
```
每个语句都是用一个值为`thisValue`的`this`调用了`fn`。第一个通过`fn`的`.call`方法调用的，JavaScript显式的用`.call`给`fn`设置了`this`。第二个是用call调用方式调用了`Function.prototype.call`方法，将`fn`这个`this`值作为第一个参数。当然，传递给`.call`的第一个参数，在这个情境中，`thisValue`是`fn`的`this`值，这个`fn`又是`.call`的`this`的值，但这不是最有趣的部分。

这才是最有趣的部分：如果你可以用`Function#call`给一些函数设置`this`值，你同样可以用`Function#bind`函数绑定`this`值。当使用`.call`的`.bind`方法时，你就能创建一个函数很自然的接收一个`this`值作为它的第一个参数。
```
var callBoundFn = Function.prototype.call.bind(fn);
callBoundFn(thisValue);
```
让我们让这个很长的故事再长一点...
```
// Instead of having to use slice.call every time,
var slice = Array.prototype.slice;
var args = slice.call(arguments);

// You can just use slice without .call by binding slice TO call!

var slice = Function.prototype.call.bind(Array.prototype.slice);
var args = slice(arguments);

// Although I'd probably call it something like toArray.
var toArray = Function.prototype.call.bind(Array.prototype.slice);
var args = toArray(arguments);
```
如果你想给`.apply`方法绑定一个函数，你也可以这么做...
```
// Math.max and Math.min take any number of arguments
Math.max(15, -10, 0 , 5, 10);      // 15

// But they don't know how to handle arrays.
Math.max([15, -10, 0 , 5, 10]);    // NaN

// So, let's fix that :)
var maxArray = Function.prototype.apply.bind(Math.max, null);
maxArray([15, -10, 0 , 5, 10]);   // 15
```
很酷，是不是？

### Bonus WTF
我已经说过我不知道什么时候停下来。最近的一次谈话使我想起了一些相关的要点，所以我挖掘出了这个看起来很奇怪的东西。
```
// OOP (yes, .blink exists)
'CHAI'.blink()                       // 'CHAI'

// Call invavation.
String.prototype.blink.call('CHAI')  // same

// $ always makes code look awesome. Right, jQuery?
var $ = Function.prototype.call;

// Very explicit call invocation.
$,call(String.prototype.blink, 'CHAI')   // same, etc...

// Very, very explicit call invocation, aka...call invo-cursion?
$.call($,$,$,$,$,$,$,$,$,$,$,$, String.prototype.blink, 'OHAI')
// 
```