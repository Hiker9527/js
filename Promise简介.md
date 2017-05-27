## Promise简介

### 1. 什么是Promise
`Promise`是抽象异步处理对象以及对其进行各种操作的组件，是异步编程的一种解决方案。`Promise`对象表示一个异步处理的最终完成(或失败)以及其产生的值，并提供统一API对其进行各种操作。

在javascript中，大多数情况下都会利用回调函数来解决异步处理，但是基于回调函数的异步处理没有统一的规则，写法因人而异。而Promise则把类似的异步处理对象和处理规则进行规范，必须按照统一的接口来编写。

由于Promise采用统一的接口，就形成了基于接口的各种各样的异步处理模式，从而可以将复杂的异步处理轻松的进行模式化。

### 2. Promise简介
+ Promise的状态
 
Promise对象有以下三个状态: `Pending`(执行中)、`FUlfilled`(已完成)和`Rejected`(已失败)。由于`Promise`状态`[[PromiseStatus]]`是在内部定义的，没有公开的API访问，所以`Promise`对象的状态只能由异步处理的结果决定，一旦从Pending转换为Fulfilled或者Reject之后，这个`promise`对象的状态就不会在发生任何变化。

(加插图)
### 用`Promise`编写代码

首先我们要创建一个`promise`对象。想要创建一个`promise`对象，可以使用`new`操作符来调用`Promise`的构造器来进行实例化。
```
var promise = new Promise(fn);
```

`Promise`构造器接受一个函数`fn`作为参数，并且在`fn`中指定异步处理，如果结果正常的话就调用`resolve(处理结果值)`，如果结果错误的话，就调用`reject(Error对象)`。它们是两个函数，由JavaScript引擎提供，不用自己部署。

`resolve`函数的作用是将Promise对象的状态从Pending转换为FulFilled，通常在在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject`函数的作用是，将Promise对象的状态从Pending变为Reject，在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

类似的，`Promise.reject(error)`也会返回一个状态为Reject的promise对象。

Promise对象创建完成以后，就可以使用`then`方法来指定FulFilled状态和Reject状态的回调函数了。
```
promise.then(onFulfilled(value), onRejected(error));
```
`then`方法可以接受两个回调函数作为参数，并返回一个全新的Promise对象。`onFulfilled`函数在promise对象状态变为Fulf时调用，`onFulfilled(value)`接受的参数就是在异步处理成功时`resolve(value)`传递出来的值。类似的，`onRejected`是promise对象状态变为Reject时的回调函数。如果只传一个回调函数则默认是`onFulfilled`。如果只想处理异步操作失败的情况，那么就给`then`方法的第一个参数传入`undefined`,使用方法`then(undefined, onRejected)`。

下面是一个简单的例子
```
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```
`timeout`返回一个Promise实例，过了指定时间(`ms`)以后，调用`resolve`方法将promise的状态设置为FulFilled，此时就触发了后面`then`方法中的回调函数。


+ catch

在处理promise发生的错误时，还可以也用promise的另一个方法`catch(error)`来指定错误回调函数;
其实`catch`只是`then(undefined, onRejection)`别名，两种方法完成的功能都是一样的。

```
var p = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
});

p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同于
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
  ```
我们推荐总是用`catch`方法来指定错误处理回调函数。
下面这个例子中用两种方法产生不同的结果：
```
function throwError(value){
    //抛出异常
    throw new Error(value);
};

//onReject不会被调用
function badMain(onRejected) {
    return Promise.resolve(42).then(throwError, onRejected);
}

//有异常发生时onReject会被调用
function goodMain(onRejected) {
    return Promise.resolve(42).then(throwError).catch(onReject);
}

badMain(() => console.log('BAD');
badMain(() => console.log('GOOD');
 ```
 
在`badMain`中，`then`里面指定的错误处理函数并没有捕获第一个参数`onFulFilled`中抛出的错误。因为`then`方法中的`onRejected`回调函数针对的是调用`then`方法的promise对象或者之前的promise对象，而不是`then`方法中的`onFulFilled`回调函数。所以在这个`then`中产生的异常，只有在后面的`catch`中才能捕获。

为了避免同步调用和异步调用同事存在导致的混乱，Promise规定只能使用异步调用的方式。

### new Promise的快捷方式
除了使用`new Promise()`来创建promise对象外，我们还可以使用静态方法`Promise.resolve`和`Promise.reject`快捷创建promise对象.
#### Promise.resolve
`Promise.resolve`返回值是一个状态为FulFilled状态的promise对象。所以我们可以接着对其返回值进行`.then`的调用:
```
Promise.resolve(42).then((value) => console.log(value));
```

`Promise.resolve`方法的另一个作用就是把thenable对象转换为promise对象。thenable对象简单说就是一个类似promise的东西，指的是一个具有`.then`方法的对象。这种转换要求hthenable对象所拥有的`then`方法和promise的`then`方法具有一样的功能和处理过程。

最常见的例子就是`jQuery.ajax()`，可以使用`promise.resolve`来转换成一个promise对象
```
var promise = Promise.resolve($.ajax('./xxx.json')); // => promise对象
promise.then((value) => console.log(value));
```

`Promise.reject(error)`是与`Promise.resolve(value)`类似的静态方法，不同之处在于promise内调用的函数是`reject`而不是`resolve`。


##### Promise链式调用
前面我们已经用过`then().catch()`这种写法了，其实在promise中可以把任意多的方法连在一起形成一个方法链。
```
function taskA() {
    console.log('TaskA');
};

function taskB() {
    console.log('TaskB');
};

function onRejected(error) {
    console.log('Catch Error: ', error);
};

function finalTask() {
    console.log('Final Task');
};

var promise = Promise.resolve(42);
promise.then(taskA)
       .then(taskB)
       .catch(onRejected)
       .then(finalTask);
```
上面的代码执行流程应该为  ==taskA -> taskB -> finalTask==;因为没有异常产生，所以`onRejected`不会被执行；

但是如果在`taskA`或者`taskB`中抛出错误，执行顺序又回是怎样
```
function taskA() {
    console.log('TaskA');
    throw new Error("throw error @ taskA");
};
```
此时的执行顺则变为==taskA -> onRejected -> finalTask==, 当 `taskA`抛出异常时，返回一个状态为 `Rejected`的promise对象，后面的`then`方法中并没有指定错误处理的回调函数，然后被`catch`捕获，执行`onRejected`函数。

那么如何把上一个`then`方法的回调函数中产生的结果传递给下一个`then`的回调函数。其实很简单，一个回调函数中返回的值会自动传递给下一个`then`的回调函数,看一个例子：
```
function double(value) {
    return value * 2;
}

function increment(value) {
    return value + 1;
}

var promise = Promise.resolve(1);

promise.then(increment)  //1 + 1
        .then(double);   // (1 + 1) * 2
// 4
```

有了Promise的链式调用，我们就可以同时处理多个异步了，
`promise.then().then().then()`。但是如果处理的大量异步的时候，就会有很多的`then`调用，这种写法看起来比较晦涩。

为了应对这种需要对多个异步调用进行统一处理的场景，promise提供了`promise.all`和`promise.race`这两个静态方法


#### promise.all

`Promise.all`接收一个promise对象数组作为参数，当这个数组中的所有Promise对象全部变为FulFilled的时候，才会去调用`then`方法，只要其中一个promise对象状态变为Reject，这个对象就会调用后面的`catch`方法。
```
function getJSON(URL) {
    return new Promise(function(resolve, reject) {
        var req = new XMLHttpReques();
        req.open("GET", URL, true);
        req.onload = function(){
            if(req.status === 200){
                resolve(req.responseText);
            } else {
                reject(new Error(req.statusText));
            }
        };
        
        req.onerror = function(){
            reject(new Error(req.statusText));
        };
        req.send();
    });
}

var p = Promise.all([p1, p2, p3]);

// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```
上面的代码中6个promise会同时开始执行，并且后面的`then`得到的promise数组的执行结果和传入`Promise.all`的promise数组的顺序是一致的。

#### promise.race

与`Promise.all`类似，`Promise.race`也是对多个promise对象进行处理的方法，接收一个promise对象数组。

`Promise.all`是爱所有的promise对象都变为FulFilled或者Rejected状态之后才会继续进行后面的处理，与之相对的`promise.race`只要有一个promise对象率先进入FulFilled或者Reject状态就会进行后面的处理。
```
var winnerPromise = new Promise(function(resolve){
        setTimeout(() => {
            console.log('this is winner');
            resolve('this is winner');
        }, 4);
    });

var loserPromise = new Promise(function(resolve){
        setTimeout(() => {
            console.log('this is loser');
            resolve('this is loser');
        }, 1000);
    });

//第一个promise变为resolve后程序停止
Promise.race([winnerPromise,loserPromise]).then((value) => console.log(value));
// => 'this is winner'
```
运行上面的代码，winner和loser promise对象的`setTimeout`方法都会执行完毕。也就是说，`promise.race`在第一个promise对象变为FulFilled后，并不会取消其他的promise的执行。

### Promise实现类库

由于并不是所有的浏览器都支持Promise对象，所以要在一些不支持Promise的环境中使用Promise规定的方法，就需要通过Polyfill类库来支持。
主流的Polyfill类库有:

 
[jakearchild/es6-promise](https://github.com/jakearchild/es6-promise)，一个兼容ES6 Promise的Polyfill类库

[yahoo/ypromise](https://github.com/tildeio/rsvp.js)，这是一个独立版本的YUI的Promise Polyfill,j具有和ES6 Promise的兼容性。

[getify/native-promise-only](getify/native-promise-only),以ES6 Promises的polyfill为目的的类库，严格按照ES6 Promises的规范设计。

#### Promise.prototype.done
Promise实现类库中都提供了`Promise.prototype.done`的方法，使用起来也和`then`一样，但是不会返回promise对象。由于`done`不会返回promise对象，所以它只能出现在方法链的最后。`done`中发生的异常会直接抛给外面，所以就算忘记调用`catch`处理错误，异常也不会被遗漏。

下面是`Promose.prototype.done`的简单实现:
```
Promise.prototype.done = function(onFulfilled, onRejected) {
    this.then(onFulfilled, onRejected)
        .catch(function(reason) {
            setTimeout(() => {throw reason}, 0);
        });
    
}
```
从上面代码看出，不管怎样，`done`都会捕捉到任何可能出现错误，并抛给全局。

#### Promise.prototype.finally
`finally`方法用于指定不管Promise对象最后状态如何，都会执行的操作。它与`done`方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

简单实现
```
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```
不管前面的Promise是Fulfilled还是Rejected状态，都会执行回调函数。

> 参考资料
>
> + Javascript Promise 迷你书
>
> + 阮一峰老师的[ECMAScript 6入门](http://es6.ruanyifeng.com/#docs/promise)



