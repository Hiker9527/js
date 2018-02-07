## 深入理解React `setState()`

- #### setState的执行过程

在学习React过程中`setState()`一直令人非常迷惑，在React的官方文档中有这样一句关于`setState()`的描述 **==状态更新可能是异步的==**，那么问题来了，它到底是异步的还是同步？什么时候是异步的？什么时候又是同步的？这几个问题困扰了很久。

先来一段`setState`的经典代码:
```
 class Root extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       count: 0
     };
   }
   componentDidMount() {
     let me = this;
     me.setState({
       count: me.state.count + 1
     });
     console.log(me.state.count);    // 打印出0
     
     me.setState({
       count: me.state.count + 1
     });
     console.log(me.state.count);    // 打印出0
     
     setTimeout(function(){
      me.setState({
        count: me.state.count + 1
      });
      console.log(me.state.count);   // 打印出2
     }, 0);
     
     setTimeout(function(){
      me.setState({
        count: me.state.count + 1
      });
      console.log(me.state.count);   // 打印出3
     }, 0);
   }
   render() {
     return (
       <h1>{this.state.count}</h1>
     )
   }
 }
 ```
我们先以异步的思路来尝试解释上面几个`console.log()`的打印：

由于`setState`是异步的，所以前两个console.log()打印的值依然是`me.state.count`的初始值`0`,第三个`setState`由于是放在`setTimeout`中的，此时前两个`setState()`已经执行完毕，这个`me.state.count`的值应该是`1`了。按照异步的推论，第三个`console.log`应该打印出`1`才对，但实际却打印了`2`，这个`setState`怎么又是同步的了。从这里可以验证文档里的那句话 **==状态更新可能是异步的==**。

要彻底弄明白`setState`在什么情况下是同步的，又在什么 时候是异步，必须搞清楚`setState`的过程。

当组件在执行`React.render`后，会对这个组件的批量更新的状态初进行始化，让组件进入批量更新的状态。然后在一个React的 **==事务==** 实例中开始装载组件。关于事务的概念，简单来说就是对一个方法进行包装，在方法开始执行之前先进行一系列的初始化，然后开始执行方法，当方法执行完毕后再进行一系列的close操作。

组件在调用装载完成之后会调用`componentDidMount`（注意，此时只是装载完成，事务并没有结束）,在这里调用了`setState`。`setState`方法主要做了两件事，第一，先把我们传给`setState`的新的状态添加到内部的一个`_pendingStateQueue`队列中保存起来，然后执行一个`enqueueUpdate`方法。第二，如果我们传入了回调函数，就把回调函数也放入一个叫`_pendingCallbacks`的队列中。

`enqueueUpdate`方法会判断当前的是否是批量更新的状态，如果是在批量更新的状态，就把当前的组件添加到`dirtyComponents`，在将来的某个方法中被批量更新。
至此,`setState`方法已经执行完毕，但是并没有更新state，所以上面代码中前两个`console.log`打印的还是最初的`this.state.count`。`componentDidMount`在这里也执行完毕了。

那么，保存在`_pendingStateQueue`队列中的state是什么时候被更新的。我们前面说的事务会在方法执行完成后执行一系列的close操作，state的更新正是在close中完成的，并且重置了组件批量更新的状态，由`true`重置为`false`。

从这个过程中可以看出`setState`的异步并不是通过`setTimeout`此类的方法实现，而是通过方法的执行顺序来实现的。

梳理清楚前两个`setState`为什么是异步的，接下来再看为什么`setTimeout`里的`setState`就是同步的。

毋庸置疑`setTimeout`中执行的函数一定是异步的，而我们前面所说的事务中一系列方法，不管是初始化操作还是正式的方法还有close操作，都是同步方法。传给`setTimeout`的回调一定是在事务结束之后才开始执行的。这里的`setState`同样也会开启一个新的事务。这个事务是同步的，`console.log`并不包括在这个事务中，state已经更新完成，所以这里会打印`2`。

我们已经清楚了`setState`更新状态的过程，总结下来就是：当与`setState`在同一个React事务中的时候会表现为异步，因为状态更新总在事务完成的时候。当`setState`在一个单独的事务中的时候，就表现为同步。判断什么时候`setState`是一个单独事务的简单方法就是，如果是用不受React管理的方式调用的，就是单独的事务，否则就在同一个事务中。

```
 class Root extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       count: 0
     };
     this.onClick.bind(this);
   }
   
   componentDidMount() {
     let me = this;
     React.findDOMNode(this)
       .addEventListener( "mousedown", this.updateState );
   }
   
   updateState() {
       this.setState({
         count: this.state.count++
       });
       
       console.log(this.state.count);
   }
   
   onClick() {
       this.updateState();
   }
   
   render() {
     return (
       <h1 onClick={this.onClick} >{this.state.count}</h1>
     )
   }
 }
 ```
 
上面的示例中，当由`onClick`调用的`setState`会表现为异步，但是由`mousedown`事件调用的`setState`包括我们之前的`setTimeout`则都表现为同步。


- #### 在setState中使用当前状态
那么如果我们想在setState中使用`this.state`，还有没有办法，答案是有的。

`setState`方法除了能传一个新的状态对象作为参数，还能传入一个返回对象的函数。下面的代码中我们假设初始状态`count`为`0`:
示例一：
```
 class Root extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       count: 0
     };
   }
   componentDidMount() {
     let me = this;
     // me.state.count === 0
     me.setState({
           count: me.state.count + 1
       });
     // _pendingStateQueue中保存{ count: 1 }
        
     // me.state.count === 0
     me.setState({
           count: me.state.count + 1
       });
     // _pendingStateQueue中仍然保存{ count: 1 }
   }
   render() {
     return (
       <h1>{this.state.count}</h1> // 显示1
     )
   }
 }
```
示例二：

```
 class Root extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       count: 0
     };
   }
   componentDidMount() {
     let me = this;
     me.setState((state, props){
     // 这里的state是当前组件的state
       return ({
           count: state.count + 1
       })
       
       // _pendingStateQueue中保存{ count: 1 }
     });

     me.setState((state, props){
     // 这里的state是_pendingStateQueue中保存{ count: 1 }
       return ({
           count: state.count + 1
       })
       // _pendingStateQueue中保存{ count: 2 }
     });

   }
   render() {
     return (
       <h1>{this.state.count}</h1> // 显示2
     )
   }
 }
```
文档中说 **==状态更新可能是异步的==**， 为什么给`setState`传入函数就能同步的获取当前的状态。
在传给`setState`的函数中，React会给这个函数传入两个参数`state`和`props`，这两个参数很容易理解，一个是组件的状态，一个属性。注意第一个参数`state`并不是`this.state`，这个参数是由React内部提供的state，回想一下前一节说过React会合并`setState`操作进行批量更新状态，每次调用`setState`后组件新的状态都会保存在`_pendingStateQueue`。这个`state`参数就是保存在`_pendingStateQueue`中的`state`。

看看源码马上就会明白，在执行批量更新操作时会执行这样一个方法:
```
_processPendingState: function (props, context) {
    var inst = this._instance;    // _instance保存了Constructor的实例，即通过ReactClass创建的组件的实例
    var queue = this._pendingStateQueue;
    var replace = this._pendingReplaceState;
    this._pendingReplaceState = false;     this._pendingStateQueue = null;
 
    if (!queue) {
      return inst.state;
    }
 
    if (replace && queue.length === 1) {
      return queue[0];
    }
 
    var nextState = _assign({}, replace ? queue[0] : inst.state);
    for (var i = replace ? 1 : 0; i < queue.length; i++) {
      var partial = queue[i];
      _assign(nextState, typeof partial === 'function' ? partial.call(inst, nextState, props, context) : partial);
    }
 
    return nextState;
},
```
这里的`replace`先忽略，认为是`false`。`nextState`的初始值是当前组件状态的一个copy，在下面的`for`循环中，每次都会按顺序从`_pendingStateQueue`中取出一个状态，与`nextState`合并，最后批量更新。`partial.call(inst, nextState, props, context)`就是当`setState`的参数为函数的时的执行，`nextState`也就时这个函数中`state`参数就是`_pendingStateQueue`队列前面几次合并后的状态。

- ### setState回调函数
`setState`还接收第二个可选参数，作为状态更新完成后的回调函数。
```
setState({
    count: this.state.count++
}, () => {
    console.log('状态更新完成')
})
```
如果想要在状态更新完成之后再执行一些操作，就可以利用`setState`的第二个参数。他可以保证方法的执行顺序，不用考虑当前的`setState`是不是同步，是不是独立的事务。


- ### 避免循环调用
禁止在`shouldComponentUpdate`或`componentWillUpdate`方法中调用`setState`。
```
    shouldComponentUpdate------->------ stetState
         |                               |
         |-----<----updateComponent--<---|
```
如果在`shouldComponentUpdate`中调用`setState`，由于状态发生了变化React就会再次执行`shouldComponentUpdate`，而`shouldComponentUpdate`又会调用`shouldComponentUpdate`，这三个方法就会互相调用，这样会造成循环调用，最后导致浏览器内存耗尽而奔溃。

