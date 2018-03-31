JavaScript是一种面向对象的语言，但是在JavaScript中并不存在类似与典型的面向对象语言种的（class）的概念。所以JavaScript也就没有所谓的“子类”和“父类”，也没有在语言层面明确规定继承的机制。JavaScript的继承机制是通过构造函数（constructor）和原型链（prototype chains）来实现的。

#### prototype的由来
我们先来解释一下JavaScript中为什么会有prototype这个概念。

JavaScript设计初衷只是用来在页面的表单发送给服务器之前，对表单进行简单的验证，当时的设计者并没有打算将这种语言设计的跟Java或是C++一样重。JavaScript没有采用“类”的概念，而是使用了构造函数（constructor）来创建对象实例。但是使用构造函数生成对象有一个明显的缺点就是没有办法共享属性和方法，每生成一个对象实例就会完全创建构造函数中所有的属性和方法，包括那些所有实例都完全一样的属性和方法。这样的话属性之间没有办法共享这些相同的数据，也造成了内存的浪费。

为了实现对象实例间属性和方法的共享，就在构造函数中设置了一个prototype的属性。这个属性指向了一个对象，存放了对象实例要共享的属性和方法，这个对象被称为“原型对象”。当用new命令调用构造函数生成一个对象实例后，这个实例就能引用到构造函数的prototype属性指向的那个对象，也就是该构造函数所有对象实例的原型对象。如此就实现了属性的共享。

JavaScript继承机制的设计思想就是基于prototype的原型链。

但是我们前面说过，JavaScript没有明确规定继承机制，继承细节并非完全由解释程序处理，所以开发人员可以实现自己的继承方式。

下面介绍几种JavaScript继承的实现方式：

#### 原型链继承

构造函数的prototype对象的所有属性和方法都可以传递给该构造函数生成的的所有对象实例。prototype对象本质上也是一个对象，所以它也有自己的原型对象，这样就形成了一个原型链。我们可以利用prototype对象的这个特性来实现继承。
```
function Super() {
  this.val = 1;
  this.arr = [1];
}

function Sub() {};
Sub.prototype = new Super();

const sub1 = new Sub();
const sub2 = new Sub();

sub1.val = 2;
sub1.arr.push(2);

console.log(sub1 instanceof Super) // true
console.log(sub1 instanceof Sub)  // true

console.log(sub1.val); // 2
console.log(sub2.val); // 1

console.log(sub1.arr); // 1,2
console.log(sub2.arr); // 1,2
```
这种继承方法的核心就是将超类的对象实例作为子类的原型对象。这样所有的子类就可以共享构父类实例的所有属性和方法。

原型链继承的优点：
1. 实现简单，容易理解。实现继承实际上我们只用了一行代码，就是把父类的实例赋值给子类构造函数的prototype属性。
2. 完全基于原型链的继承，继承关系清晰，子类对象实例的即是子类构造函数的实例，同时也是超类构造函数的实例。
3. 子类的prototype对象是超类的实例，子类的实例同样也可以访问到超类的原型对象，在超类的圆形对象上添加新的属性和方法，所有子类的实例也都可以访问到。

但是这种简单的继承实现也有很大的缺点：
1. 子类的所有实例都会共享原型对象中的引用属性。看上面的示例，我们给`sub1.arr`属性添加了一个元素，但是`sub2.arr`也发生了变化，有时候这并不是我们想要的。
2. 创建子类实例时，无法向超类的构造函数传参。当我们调用子类的构造函数生成子类实例的时候，超类已经被实例化了，我们只能在指定子类的prototype的时候给超类传参。
3. 无法实现多继承。子类的原型对象只能指向一个超类的实例，不能同时指向多个。

#### 借用构造函数
使用new命令调用构造函数创建实例对象时，首先会创建一个空的对象，然后把这个空的对象赋值给构造函数内部的this。最后返回这个新创建的对象。构造继承的原理就是将子类的this作用在超类的构造函数上，此时超类只是被当作一个普通的函数调用。
```
function Super(val) {
  this.val = val;
  this.arr = [1];
}

function Sub(val) {
 Super.call(this, val);
 this.sayHello = function(name) {
   console.log(`Hello ${name}`);
 };
}

const sub = new Sub(1);

console.log(sub.val); // 1
sub1.sayHello('Jone'); // Hello Jone
console.log(sub instanceof Super) // false
console.log(sub instanceof Sub)  // true
```
构造继承的优点：
1. 没有子类实例共享超类引用属性的问题。这种继承方式没有使用prototype对象自然没有原型对象引用属性共享的问题。
2. 调用子类构造函数时可以给超类传递参数。超类的调用是在子类构造函数内，就可以把子类的参数传给超类执行。
3. 可以实现多继承。只需在子类中调用多个超类就可以实现多继承。

缺点：
1. 子类实例对象只是并不是超类的实例，只是子类构造函数的实例。因为我们调用的只有子类的构造函数，看似我们调用了超类构造函数，不过只是当作普通函数调用，本质上还是子类的构造函数。
2. 只继承了超类的实例属性，不能继承原型属性和方法。
3. 浪费资源。每次生成一个新的子类实例对象，就会完全生成一个新的对象，包括超类中的属性方法，但是这些属性和方法没有共享。

#### 组合继承
原型链继承和构造函数继承都有各自的优缺点，组合继承就是结合上面两种继承方法，达到优缺点互补的目的。
```
function Super() {
  this.val = 1;
  this.arr = [1];
}

Super.prototype.fun1 = function() {};
Super.prototype.fun2 = function() {};

function Sub() {
  Super.call(this);
  // 子类实例属性
}

Sub.prototype = new Super();
```
在定义超类构造函数的时候，我们把不想共享的属性放在构造函数内，希望共享的属性和方法放在构造函数的prototype对象上。在子类的构造函数中用上面的借用构造函数继承的方式调用超类构造函数，并创建一个超类的实例赋值给子类构造函数的prototype属性。

这种方式的优点：
1. 避免了不希望的引用属性共享问题。
2. 创建子类实例对象时可以给超类传参。
3. 子类实例可以共享原型对象。
但是这里还有一点问题就是，子类构造函数的prototype对象上会一份多余的超类实例属性。我们说放在超类构造函数体内的实例属性并不是我们希望共享的属性。因为我们在用new命令调用子类构造函数创建子类实例对象时，这部分超类的实例属性会变成子类的实例属性，就会覆盖原型对象上的属性。

#### 原型式继承

```
function object(o) {
  const F = function(){};
  F.prototype = o;
  return new F();
}

function Super() {
  this.val = 1;
  this.arr = [1];
}

const sup = new Super();

const sub = object(sup);
sub.attr1 = 1;
sub.attr2 = 2;
// ...
```
用object方法创建一个没有任何实例属性的新对象，这个对象的原型对象是传给object方法的那个对象，然后增强这个新创建的对象（或者可以这么说，复制一个超类的实例，然后添加新的属性）。仔细想想这种方式跟原型链继承区别并不是很大，把原型链继承中的子类构造函数封装在object函数中，去掉所有的实例属性和方法，就变成了原型式继承。

优点：
1. 不用再定义一个子类的构造函数，直接从已有的对象衍生新对象。我们在上面的代码中虽然定义了一个超类Super构造函数，但这并不是必须的，完全可以用一个已经存在普通的对象代替。

缺点：
1. 这种继承方式与原型链的继承没有本质的区别，所以也存在原型链继承中的缺点。
#### 寄生继承
```
function object(o) {
  const F = function(){};
  F.prototype = o;
  return new F();
}

function Super() {
  this.val = 1;
  this.arr = [1];
}

function createAnother(origin){
  // 创建一个新对象
  const clone = object(origin);
  // 增强
  clone.say = function(){
    alert('hi')
  }
  // 返回
  return clone;
}
const sub = createAnother(new Super());
```
寄生式继承可以看作是原型式继承的加强版。只不过是把创建增强的这个过程封装在了一个函数中。一句话概括就是创建一个新的对象，增强后将这个对象返回。

#### 寄生组合继承
寄生组合就是利用寄生继承的优点修复组合继承的那点不足。
```
function object(o) {
  const F = function(){};
  F.prototype = o;
  return new F();
}

function Super() {
  this.val = 1;
  this.arr = [1];
}

Super.prototype.fun1 = function() {};
Super.prototype.fun2 = function() {};

function Sub() {
  Super.call(this);
  // ...
}

const proto = object(Super.prototype);
Sub.prototype = proto;
Sub.prototype = Sub;

const sub = new Sub();
```
寄生组合继承用原型式继承中的object方法创建了一个没有任何实例属性的空对象，这个对象的原型对象是超类构造函数的prototype对象。把这个新创建的对象赋值给子类构造函数的prototype属性，就去掉了组合继承中多出来的那一份超类的实例属性。最后我们把子类构造函数prototype对象上的constructor属性指向子类的构造函数。因为proto这个对象的constructor引用的是超类的构造函数。

寄生组合式继承基本上可以算的上比较完美的继承实现了。由于结合了几种继承方式，理解起来没有另外几种容易。虽然寄生组合式继承，比较完美，但是用的最多的还是组合继承。