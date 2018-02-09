Redux的作者对于中间件的解释是“It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.”大致的意思是说它在dispatch了一个action后和在reducer接收到这个action之前提供了一个三方的扩展。它提供了一个分类处理action的机会，在middleware中拦截所有的action，然后改变这些action的行为。

#### 一个场景
点击一个button后分发一个action，reducer响应了这个action，修改了state，view监听到了state发生了更新就会重新渲染页面。但是如果我们需要记录每次action的信息应该怎么办？

#### 满足单个场景的中间件
首先想到的就是在button的点击事件中加入记录点击的功能。比如下面的代码：
```
function onClick(arg) {
    const action = actionCreate(arg);
    console.log('before dispatch');
    store.dispatch(action);
    console.log('dispatch done');
}
```
但是这样的场景很多，在每次调用`dispatch`之前都要加入这么一段重复的代码，显然不是我们想要的。

那么我们就要想办法只需要在一个地方写一遍就好了。既然是在每次调用`dispatch`的时候要加入这些代码，何不直接把这些代码写在`dispatch`中，这样每次在分发action的时候就会自动打印这些日志了。接下来我们重写dispatch，在执行原有的store.dispatch的同时加入一些别的功能:
```
let next = store.dispatch;
store.dispatch = function(action) {
    console.log('before dispatch');
    const result = next(action);
    console.log('dispatch done');
    return result;
}
```
看起来没什么问题了，不过我们还是要稍微加工一下，用一个函数题包起来，需要的时候执行这个函数就好了：
```
function logger(store) {
  let next = store.dispatch;
  store.dispatch = function(action) {
    console.log('before dispatch');
    const result = next(action);
    console.log('dispatch done');
    return result;
  }
}
// 这样使用
logger(store)；
```
看起来差不多是我们想要的样子了，但是如果我们又想在加入别的功能，要怎么办，继续往上面的方法里添加代码？显然不是的，我们应该把不同的功能分开，像一个个的包一样，按需引用。
```
logger(store)；
anotherLogger(store)；
```
#### 实现链式调用
这里有个疑问，中间件覆盖了前面的中间件，为什么前面的中间还会生效。如果你仔细观察就能发现，后面的store.dispatch都指向了前一个中间件里面的一个匿名函数，后面的中间会再将前面的store.dispatch（也就是那个匿名函数）包装到新的匿名函数里面。最后的效果就是这些中间件按调用顺序倒序执行。比如上面的示例代码中，先执行`anotherLogger`再执行`logger`。

如果加载的中间件比较多的时候，我们应该实现一个执行器，让代码变的更优雅和可理解。继续修改代码。
```
function logger(store) {
    // 这个next是上一个中间件中的dispatch方法
    let next = store.dispatch;
    return (action) => {
        console.log('before dispatch');
        let result = next(action);
        console.log('dispatch done');
    }
}

function applyMiddleware (store, middlewares) {
    middlewares = middlewares.slice();
    // 反转一下以便让中间件按我们传入的顺序执行
    middlewares.reverse();
    
    middlewares.forEach(middleware =>
        store.dispatch = middleware(store)
    )
}
// 这样用
applyMiddleware(store, [logger, anotherLogger])
```
#### 真正的Redux中间件
这个时候我们离真正的Redux中间件只有一步之遥了。我们先看看标准的Redux中间件是什么样的：
```
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```
唯一的区别就是在标准的Redux的中间件直接把next作为参数传了进去，并不是在store中获取的。这样我们就不用在每次调用中间件的时候去改写store.dispatch了。避免所谓的猴子补丁（Monkeypatching）的做法。

Redux的插件借鉴了Koa的middleware的思想，支持插件可组合、自由插拔。


### Redux middleware机制
Redux提供了applyMiddleware方法加载插件，实现了插件的组合、自由插拔的机制，源代码很少却是非常精炼，非常值得研究。
```
import compose from './compose';

export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    let store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
虽然只有二三十行代码，但是信息量很大。下面我们就结合代码深入剖析applyMiddleware的原理。

applyMiddleware这个方法是一个curry化的函数。curry化是函数式编程的一个概念，就是用匿名单参函数实现多个参数的方法。curry函数具有保存参数和延迟执行的特性，在函数式编程中非常有用。
```
// ...
let store = createStore(reducer, preloadedState, enhancer)
let dispatch = store.dispatch
let chain = []

var middlewareAPI = {
    getState: store.getState,
    dispatch: (action) => dispatch(action)
}
```
不管前面几行，我们假设applyMiddleware被正确执行了。函数体中首先调用`createStore`的方法和前面传入的参数创建了一个store。然后用一个变量保存了store.dispatch，有定义了一个数组变量chain。还定义了一个有`getState`和`dispatch`的对象`middlewareAPI`。这个对象包含了store的两个核型的方法，其实也就是一个不完全的store。


```
// ...
chain = middlewares.map(middleware => middleware(middlewareAPI))
dispatch = compose(...chain)(store.dispatch)
// ...
```

这两行代码就是整个`applyMiddleware`方法的重点所在。

第一行用`map`方法遍历所有的中间件，然后把上面定义好的middlewareAPI也就是这个不完全的store传给每个middleware执行一次。

这里再列出Redux中间件示例：
```
const logger = store => next => action => {
  console.info('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```
中间件的store参数就是`middlewareAPI`这个对象。在传入store这个参数并执行后中间件继续返回第二层的匿名函数。所有的中间件都在一个闭包中，它们都会引用同一个store。`chain`这个数组中保存的就是一个以匿名函数为元素的数组。可以这样表示`chain = [f1,f2, ..., fn]`。
此时fn的格式应该是这样的：
```
next => action => {
  ...
  // 这里可以访问store，且所有中间件共享一个store
  let result = next(action)
  ...
  return result
}
```
第二行是最精华的一行代码。
```
dispatch = compose(...chain)(store.dispatch)
```
`compose`这个方法在函数式编程中并不陌生，作用就是把若干
方法组合成一个新的方法。它把chain中的所有匿名函数组合在一个返回一个新的函数。关于`compose`的实现有很多，下面介绍一下Redux中的实现。
```
function compose (...funcs) {
    return args => funcs.reduceRight((composed, f) => f(composed), args)
}
```
compose返回一个匿名函数，这个匿名函数接收一个参数作为`reduceRight`的初始值，然后迭代`funcs`这个数组。`composed`的是上一次执行的返回值，`f`是当前的数组元素，把上次的返回值`composed`作为`f`的参数执行这个函数。

在当前的代码中`funcs`就是`chain`，`store.dispatch`就是`args`。展开的话就是下面的样子：
```
dispatch = f1(f2(...fn(store.dispatch)...))
```
现不要管嵌套了多少层，回想一下fn的格式
```
next => action => {
  ...
  // 这里可以访问store，且所有中间件共享一个store
  let result = next(action)
  ...
  return result
}
```
f1已经得到了一个参数就是`next`,所以它的返回值就是
```
action => {
  ...
  // 这里可以访问store，且所有中间件共享一个store
  let result = next(action)
  ...
  return result
}
```
这就是一个典型的dispatch。当我们执行`dispatch(action)`的时候这个嵌套的匿名函数就是从外向里依次调用fn，也就是中间件中的`next`，到达最里面一层再由内向外依次返回到最外一层所有的中间件都会被按顺序执行一遍。

![image](https://github.com/Hiker9527/js/blob/master/static/redux-middleware.png)
这里我们再回头来看看为什么`middlewareAPI`中的`dispatch`方法是一个匿名函数而不是`store.dispatch`。因为我们要保证在`applyMiddleware`执行完成之后，所有中间件中的`store.dispatch`都是最新的而不是原始的`store.dispatch`。应为`middlewareAPI.dispatch`是一个闭包，这个闭包维持了对最外层作用域中`dispatch`的引用。当`applyMiddleware`执行完毕之后`middlewareAPI.dispatch`的这个闭包中的`dispatch`引用的就是最终的`dispatch`。

这就是Redux中间件的实现机制，虽然简短但是一点都不简单，函数式编程的特性在这几行代码中得到了深刻的体现。

#### 编写middleware
既然我们已经清楚了middleware的机制，那么编写一个自己的中间件就是一件很简单的事情了。比如我们熟知的redux-thunk中间件：
```
store => next => action => {
    typeof action === 'function' ?
    action(store.dispatch, store.getState) : next(action)
}
```

我们之前说过action必须是一个对象，但是在这里action有可能是函数。在action被正式传入原始的`store.dispatch`之前，先判断action是不是函数，如果是函数就执行，如果不是就调用next。