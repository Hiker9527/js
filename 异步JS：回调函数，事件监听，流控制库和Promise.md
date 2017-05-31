原文:[Asynchronous JS: Callbacks, Listeners, Control Flow Libs and Promises](http://sporto.github.io/blog/2012/12/09/callbacks-listeners-promises/)

在处理JavaScript的异步开发中有许多的方法你可以使用。这篇文章会介绍其中的四种方法和它们各自的优势。它们是回调函数，事件监听，流控制库和Promise。

### 案例
为了阐明这四种方法的的用法，我们建立一个简单的案例。

假设我们想要查找一些记录，然后运行它们最后返回运行的结果。两者(查找和运行)都是异步的。

<div style="height: 180px; overflow:hidden;">
    <img src="http://photos.foter.com/29/why-didnt-you-call-me_l.jpg" style="display: block; width: 100%; margin-top:-120px;">
</div>

## 回调函数
让我们先从回调模式开始，这是最基本也是最广为人知的处理异步编程的模式。

回调函数看起来像这样
```
finder([1, 2], function(results){
    //..do something
});
```
在回调函数模式中我们调用一个执行异步操作的函数。我们传入的参数中有一个是函数，这个函数将会在操作完成后被调用。
### 起步
为了阐明它们是怎么工作的，我们需要一对查找和运行记录的函数。在实际情况下，这些函数会发送一个AJAX请求然后返回结果，但是现在我们用超时代替。
```
function finder(records, cb) {
    setTimeout(function (){
        records.push(3, 4);
        cb(records);
    }, 1000);
}
function processor(records, cb) {
    setTimeout(function (){
        record.push(5, 6);
        cb(records);
    }, 1000);
}
```
### 使用回调函数
使用上面这些函数的代码就像下面这样：
```
finder([1, 2], function (records) {
    processor(records, function(records) {
      console.log(records);
    });
});
```
我们调用第一个函数，传入一个回调函数。在这个回调函数里面我们调用第二个函数并出入另一个回调函数。


我们还可以通过传入一个指向另外一个函数的引用来把这些嵌套的回调函数写的更加清晰。

```
function onProcessorDone(records) {
    console.log(records);
}

function onFinderDone(records) {
    processor(records, onProcessorDone);
}

finder([1, 2], onFinderDone);
```
两个案例中上面的控制台记录都是[1,2,3,4,5,6]

### 优点
+ 他们都是非常著名的模式，我们都比较熟悉而且更容易理解。
+ 非常容易就可以部署在你的库/函数中。
### 缺点
+ 嵌套回调函数会形成臭名昭著的"金字塔"，就像上面展示的一样，当你嵌套了很多层以后，代码就会变的很难阅读。但是通过像上面那样分离函数就可以很容易的解决这个问题。
+ 在给定的事件中只能传入一个回调函数，许多情况下这会是一个很大的限制。

<div style="height: 150px; overflow:hidden;">
    <img src="http://photos.foter.com/66/summer-sound-large-view_l.jpg" style="display: block; width: 100%;">   
</div>

## 事件监听
监听器也是一个很有名的模式，能流行起来很大程度上是因为jQuery和其它的DOM库。一个监听器看起来就像下面这样:
```
finder.on('done', function(event, records) {
    //..do something
});
```
我们在一个加了监听器的对象上调用一个函数。在这个函数中传入我们想要监听事件的名字和一个回调函数。'on'是这个函数许多常用的名字中的一个，你还会遇到像'bind'，'listen'，'addEventListener'，'observe'这些其它常用的名字。
### 部署
让我们为监听器示例做一些设置。很遗憾的是这些启工作要比回调函数的示例稍微复杂点。

首先，我们需要一组对象，他们将会负责查找和执行记录的工作。
```
var finder = {
    run: function (records) {
        var self = this;
        setTimeout(function () {
            records.push(3 ,4);
            self.trigger('done', [records]);
        }, 1000);
    }
}

var processor = {
    run: function (records) {
        var self = this;
        setTimeout(function () {
            records.push(5, 6);
            self.trigger('done', [records]);
        }, 1000);
    }
}
```
注意它们调用一个**trigger**的方法当任务完成之后，我会用**混入**把这个方法添加到那些对象中。'trigger'是你将会再次遇到的名字之一，其它常见的名字还有'fire'和'publish'。

我们需要一个拥有监听器行为的混入对象，在这个案例中我将依靠jQuery来实现这个：
```
var eventtable = {
    on: function(event, cb) {
        $(this).on(event, cb);
    },
    trigger: function (event, args) {
        $(this).trigger(event, args);
    }
}
```
然后把这个行为运用到我们的finder和processor对象上:
```
$.extend(finder, eventtable);
$.extend(processor, eventable);
```
很好，现在我们的对象能获得监听器和触发事件了。

### 使用监听器
使用监听器的代码很简单：
```
finder.on('done', function(event, records) {
    processor.run(records);
});
processor.on('done', function(event, records) {
    console.log(records);
});
finder.run([1,2]);
```
控制台会再次输出[1,2,3,4,5,6]
### 优点
+ 这是另一种容易理解的模式
+ 最大的优势是没有每个对象只能有一个监听器的限制，你可以想加多少监听器就加多少个。
```
finder
    .on('done', function(event, records) {
        //..do something
    })
    .on('done', function (event, records) {
        //.. do something else
    });
```
### 缺点
+ 在你自己的代码中部署要比回调函数更难一点，你可能更愿意使用一个类似于jQuery,bean.js的库。


## 流控制库
流控制库也是一个处理异步编程的不错的方法。我非常喜欢的一个库是[Async.js](https://github.com/caolan/async)。

使用Async.js的代码看起来就是下面的样子:
```
async.series([
    function(){ ... },
    function(){ ... }
]);
```
### 部署（示例 1）
我们又需要一组完成这个工作的函数，在其它例子中这些函数在实际情况下可能会发送一个AjAX请求并且返回结果。现在我们用超时代替。
```
function finder(records, cb) {
    setTimeout(function (){
        records.push(3, 4);
        cb(null, records);
    }, 1000);
}

function processor(records, cb) {

    setTimeout(function () {
        records.push(5, 6);
        cb(null, records);
    }, 1000};
}
```
#### Node continuation style(这个我不知道怎么翻译)
注意在上面的函数中，函数里面回调的方式。
```
cb(null, records);
```
如果没有错误发生，回调函数的第一个参数就是null；或者是一个error如果有error产生的话。这是一个在Node.js中常见的模式，Async.js也使用这种模式。通过这种方式Async.js和回调函数之间的流就变得非常简单。
### 使用Async
消费这些函数的代码看起来应该是这样的：
```
async.waterfall([
    function(cb) {
        finder([1,2], cb);
    },
    processor,
    function(records, cb) {
        alert(records);
    }
]);
```
Async.js会按顺序调用每一个函数当上一个函数执行完成以后。注意我们是怎样传入`processor`函数的，这是因为我们使用了**Node continuation style**。你会发现这段代码很简洁而且易于理解。


### 另一个部署(示例 2)
现在，当做前后端开发的时候不不太可能你有一个能跟踪回调函数(null, results)信号的库。所以一个更加实用的例子应该像下面这样：
```
function finder(records,cb) {
    setTimeout(function (){
        records.push(3, 4);
        cb(records);
    }, 500);
}
function prosessor(records,cb) {
    setTimeout(function (){
        records.push(5, 6);
        cb(records);
    }, 500);
}

//
asy.waterfall([
    function(cb) {
        finder([1,2], function(records) {
            cb(null, records)
        });
    },
    function(records, cb) {
        processor(records, function(records) {
            cb(null, records)
        });
    },
    function(records, cb) {
        alert(records);
    }
]);
```

### 优点
+ 通常使用控制流库的代码比较容易理解，因为它是按自然顺序(自上而下)流动的。但是回调函数和监听器不是这样的。
### 缺点
+ 如果函数的信号没有像第二个示例中进行匹配，你会觉得流控制库的可读性比较差。

## Promise
终于到了我们的最终目的地。Promise是一个非常强大的工具，但却是最难理解的。

使用promise的代码看起来是这样的:
```
finder([1,2])
    .then(function(records) {
        //.. do something
    });
```
这会非常广泛的依赖于你使用的promise库，在这个案例中我使用的是[when.js](https://github.com/cujojs/when).
>译者注：在ES6中已经原生支持promise对象了，而且有一些比较新的浏览器和Node版本已经开始支持了。

### 部署
输出的`finder`和`processor`函数是下面的样子：
```
function finder(records){
    var deferred = when.defer();
    setTimeout(function () {
        records.push(3, 4);
        deferred.resolve(records);
    }, 500);
    return deferred.promise;
}
function processor(records) {
     var deferred = when.defer();
    setTimeout(function () {
        records.push(5, 6);
        deferred.resolve(records);
    }, 500);
    return deferred.promise;
}
```
每一个函数创建了一个deffered对象并且返回一个promise.然后当收到结果的时候解决这个deferred。

### 使用promise
消费这些函数的代码看起来是这样的:
```
finder([1, 2])
    .then(processor)
    .then(function(records) {
        alert(records);
    };
```
正如你所见，代码相当短而且容易理解。这样使用的话，promise的自然流动会让你的代码更加清晰。注意我们是怎样在第一个回调函数中仅传入`processor`函数的。这是因为这个函数返回一个promise自身，所以一切都会顺畅的流动。

promise能做的还有很多
+ 它们可以像常规对象一样被传入
+ 合并成更大的promise
+ 你可以给promise加失败处理事件

### promise的更大的好处
现在如果你认为这就是promise的全部，那么你就错过了我认为的promise最大的优势。Promise有一个整洁的trick,回调函数，监听器，或者是控制流都做不到。你可以给一个promise加一个监听器甚至在当它被解决之后，这种情况下监听器就是被立刻触发，意味着你不必担心这个事件是否在你添加监听器的时候已经发生过了。这在聚合的promise中也一样有效。让我来给你演示一个例子:
```
function log(msg) {
    document.write(msg + '<br />');
}

// using promises
function finder(records){
    var deferred = when.defer();
    setTimeout(function () {
        records.push(3, 4);
        log('records found - resolving promise');
        deferred.resolve(records);
    }, 100);
    return deferred.promise;
}

var promise = finder([1,2]);

// wait 
setTimeout(function () {
    // when this is called the finder promise has already been resolved
    promise.then(function (records) {
        log('records received');        
    });
}, 1500);
```
这在浏览器中处理用户交互是一个很有用的功能。在复杂的应用中你可能不知道用户行为的顺序，因此你可以用promise来追踪用户交互。如果你感兴趣你可以看看另一篇[文章](http://sporto.github.io/blog/2012/09/22/embracing-async-with-deferreds/)

### 优点
+ 相当健壮，你可以合成一个promise，传入它们，或者在已经解决之后添加一个监听器。
### 缺点
+ 这些方法中最难理解的。
+ 当你有大量添加了监听器的聚合promise后，这将变得很难追踪。

## 结论
就只这样！这写是我认为的四种主要的处理里异步代码的方法。希望我能帮助你更好的理解它们，为你的异步需求提供更多的选择。