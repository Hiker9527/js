## 原型对象与原型链
#### 原型对象
每个函数在创建的时候都有一个prototype(原型)属性，这个属性是一个指针，指向一个对象，这个对象叫做原型对象。原型对象包含这个函数所有实例可共享的属性和方法。默认情况下这个原型对象会自动获得一个`constructor`属性，这个属性是一个指向`prototype`属性所在的那个函数的指针。

调用构造函数时，会为实例对象添加一个内部属性`[[Protoype]]`，这个属性指向构造函数**此刻**的原型对象，也就是`prototype`属性指向的那个对象。当读取一个对象的属性时，ECMAScript会首先在实例属性中查找，如果找到了对应的属性或方法就会停止查找，然后返回查找结果。如果在实例属性中没有找到，就会继续在对象的`[[Prototype]]`属性指向的那个对象中查找。在实例上的属性会屏蔽原型中的同名属性。

给原型对象添加属性或者方法可以动态的反应在所有的实例中，但是如果给构造函数的`prototype`重新赋值，就会切断构造函数与最初原型之间的联系，也就意味这切断了现有原型与之前存在的任何实例对象之间的联系，所以已经存在的实例对象无法获取新的原型上的任何属性和方法。

前面说过在对象内部有一个`[[Prototype]]`指针指向构造函数的原型对象，有部分浏览器（比如chrome和fireFox）提供了一个`_proto_`的属性来访问这属性，但是在其它浏览器中则完全不可见。不过EcmaScript提供了一个`isPrototypeOf`的方法来确定一个对象是否是一个函数的原型对象，像这样：
```
Preson.protoytype.isPrototypeOf(Person)      // true
```
同时，在 ECMAScirpt 5 中又新增了一个方法，用于获取对象`[[Prototype]]`属性指向的那个对象。
```
Object.getPrototypeOf(person1) === Person.prototype;   // true
```
`Object.getPrototypeOf()`返回的实际就是这个对象的原型。

#### 原型与 in 操作符
单独使用`in`操作符的情况下，它并不关心这个操作的属性是否是可枚举的，只要该属性可以被访问就返回`true`。但是用`for-in`这中形式的时候，返回的所有对象中可访问且**可枚举的**属性，包括实例中的，也包括原型中的属性。如果要遍历对象上所有可枚举的实例属性，还需要结合`hasOwnProperty`方法来判断这个属性是不是存在于实例中。另外 ECMAScript 5 提供了`Object.keys`，返回对象可枚举的实例属性的字符串的数组。
#### 原型链
原型对象本质上也是对象，是其它构造函数或者`Object`对象的一个实例，因此原型对象上也存在一个内部的`[[Prototype]]`指针指向它的构造函数的原型对象，所有的对象最后的`[[Prototype]]`都会指向`Object`这个对象的原型对象。如果一个属性没有在实例对象中找到或者在它的构造函数的原型对象中找到，就会继续向上查找，直到`Object`的原型对象，如果依然没有找到就返回`undefined`。

## 创建对象
就是创建一个对象的模式
#### 工厂模式
创建一个工厂函数，然后在工程函数内部实例化一个`Object`对象，给这个实例添加属性和方法，然后返回这个对象。
```
function createObj(name, age, job) {
    const o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        console.log(this.name)
    }
    return o;
}
```
工厂模式无法识别一个实例是由哪个工厂函数创建的
#### 构造函数模式
创建一个构造函数， 用`new`命令调用这个构造函数生成实例。
```
function Person(age, name) {
    this.age = age;
    this.name = name;
    this.sayName = function() {
        console.log(this.name);
    }
}
const person1 = new Person(20, 'aa');
```
用new命令调用构造函数后的的4个步骤：
1. 创建一个新的空对象；
2. 将构造函数的作用域赋值给这个对象（因此 this 就指向了这个新对象）；
3. 执行构造函数中的代码（为新对象添加属性方法）；
4. 如果没有显式的返回一个对象，那么就返回这个对象。

`instanceof`操作符可以判断一个对象是不是被操作构造函数的实例：
```
person1 instanceof Person   // true
```
也可以利用原型对象的`constructor`属性：
```
person1.constructor === Person // true
```
构造函数模式解决了对象识别的问题，但是也有缺点，所有属性和方法都会在每个实例上重新创建一遍（这个问题在工厂模式上同样存在），无法共享公共的属性和方法。

#### 原型模式
将共享的属性和方法放在构造函数的原型对象中，然后在实例中添加自己的属性和方法。
```
function Person() {}
Persion.prototype = {
    age: 20,
    name: 'aa',
    sayName: function() {
        console.log(this.name)
    }
}
const person1 = new Person();
person1.name = 'bb';
```
原型模式解决了构造函数中相同属性方法共享的问题，不过这会导致属性的值也将在所有实例中共享，在一个实例中操作共享属性，所有的实例都会受到影响。还有一个问题就是无法在创建实例的时候传递参数。
#### 组合使用构造模式和原型模式
构造模式不能共享属性和方法，原型模式将完全共享，那么就把该共享属性和方法放在原型对象中，不能共享的放在构造函数内部。
```
function Person(age, name) {
    this.age = age;
    this.name = name;
}

Persion.prototype = {
    constructor: Person,
    sayName: function() {
        console.log(this.name)
    }
}

const person1 = new Person(20, 'aa');
```
这种模式是使用最广泛的一种创建对象的模式。
#### 动态原型模式
这种模式与上面的组合模式其实是一样的，只是把给原型对象赋值的那那段代码封装在了构造函数内部，先判断方法是否已经在实例方法中定义，如果没有就添加到原型对象中：
```
function Person(age, name) {
    this.age = age;
    this.name = name;
    if (typeof this.sayName !== 'function') {
        Person.prototype.sayName = function() {
            console.log(this.name)
        }
    }
}
```
#### 寄生构造函数模式
这种模式定义类的方法与工厂模式一样，唯一不同的是在寄生构造函数模式下创建实例的时候是用`new`命令调用。
```
function Person(age, name) {
    const o = new Object();
    o.age = age;
    o.name = name;
    return o;
}
const person1 = new Person(20, 'aa');
```
要注意的是`person1.constructor`并不是`Person`，在构造函数内部返回的是`Object`的实例。不建议使用这种模式。
#### 稳妥构造函数模式
没什么说的，工厂模式类似，区别是不使用`new`和`this`。适合用在一些安全环境中。
## 继承
ECMAScript没有类的感念，继承主要是基于原型链实现的。
#### 原型链
将子类的`prototype`指向一个父类的实例。这样子类就继承了父类实例的所有方法已经原型对象上的方法和属性。这种方法的本质是重写子类构造函数的原型。由于子类的原型对象被重写成了父类的实例，所以此时子类的原型对象的`constructor`指向父类构造函数。这点要主要。
```
function SuperType() {
    // ...
}

function SubType (age, name) {
    this.name = name;
    this.age = age;
}

SubType.prototype = new SuperType();
```
缺点：
- 在创建子类实例没法给超类传参
- 子类原型对象是超类的一个实例，这个实例中的属性会在子类实例中共享
#### 借用构造函数
借用构造函数就是在子类构造函数内部用`call`调用超类的构造函数。
```
function SuperType() {
    // ...
}

function SubType(age, name) {
    SuperType.call(this);
    this.age = age;
    this.name = name;
    this.sayName = function() {
        console.log(this.name)
    }
}

const subType = new SubType(20, 'aa');
```
我们知道构造函数也是普通的函数，在子类的内部用`call`调用超类的构造你函数，将它的`this`值绑定为子类的作用域。当然我们也可以在调用的时候给超类构造函数传递参数。这种继承方式中不存在共享原型属性的问题，但是有缺点：
- 所有的属性和方法将在每个实例中创建一遍，浪费内存
- 无法判断继承关系。`subType instanceof SuperType`的值是false。`SubType`这个构造函数只是在内部以普通函数的方式调用了`SuperType`，子类的实例无超类之间并无直接的联系。
- 无法继承超类原型对象中的属性和方法。

#### 组合式继承

由于原型继承和借用构造函数的继承的这些问题，组合继承就是基于这两种继承方式上取长补短的一种实现思路。
```
function SuperType(name) {
    this.age = age;
    this.name = name;
}

SuperType.prototype.sayAge = function () {
    console.log(this.age);
}

function SubType (name, job) {
    SuperType.call(this, name);    // 继承超类的实例属性
    this.job = job;
}

SubType.prototype = new SuperType();    // 继承超类的原型属性
SubType.prototype.constructor = SubType;     // 上一步中子类的原型被指向了超类的一个实例，constructor也指向了超类，这里我们要修改回来。
SubType.prototype.sayName = function () {
    console.log(this.name);
}


const sub = new SybType('shawnwang', 'haha');
```
组合式继承解决了原型继承中属性共享的问题，也解决了借用构造函数中的问题。这种方式也是最普遍的一种继承方式。不过也有缺点：

在子类的原型上会有一份多余的超类实例属性副本。在继承超类的原型属性上我们用了一个超类的实例，这个实例中会产生一份多余的超类实例属性。

#### 原型式继承
借助原型基于已有的对象创建新的对象，让新创建的对象继承已有的对象。
```
function object(o) {
    function F() {}
    F.prototype.o;
    return new F();
}
```
借助一个临时的构造函数，然后已有的对象赋给这个构造函数的原型对象，最后返回一个实例。这种方式适合继承一个已有的对象，避免了兴师动众的创建构造函数。
#### 寄生式继承
所谓寄生式就是在一个方法内部创建一个对象，然后以某种方式增强这个对象，最后返回增强的对象。
```
function createAnother(original) {
    const clone = object(original);
    clone.sayName = function() {
        console.log(this.name)
    }
    return clone;
}
```
与原型式类似，如果在没有自定义一个类型和构造函数的情况下，这种方式实现继承是比较适合的。

实际上在 ES5 中提供了一个方法`Object.create`与我们上面的这个函数有异曲同工之妙，这个方法会创建一个具有指定原型且可选择性包含指定属性的对象，如此新的对象就会继承我们提供的作为`prototype`参数的对象，`descriptors`参数是可选的：
```
Object.create(prototype, descriptors)
```
#### 寄生组合式继承
在组合继承的模式下，在重写子类的`prototype`属性时，超类的实例属性会被多创建一次。寄生组合模式就是在重写子类的`prototype`时，并不是直接用超类的实例，而是利用超类的`prototype`属性重新创建一个新的对象。并修改这个对象的`constuctor`指向。
```
function SuperType() {
    // ...
}

SuperType.prototype.sayName = function() {
    // ...
}

function SubType() {
    SuperType.call(this/*, 其它参数 */);
    // ...
}

const prototype = Object.create(SuperType.prototype)
prototype.constructor = SubType;
SubType.prototype = prototype;

SubType.prototype.sayAge = function() {
    console.log(this.age);
}

```
这种方式在给子类的原型对象赋值的时候没有创建超类的实例。避免了在组合继承中的超类实例属性多余的一次创建。这种方式被认为是引用类型最理想的继承方式（但并不是最广泛）。
