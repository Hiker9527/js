用过React开发应用的一定对Redux不陌生但是有可能不知打Flux是什么。Redux是实现了Flux思想的一个库，由于Redux太过成功以至其知名度已经远远超过了Flux。Redux的设计非常简单，只需要几个API就能实现核型功能。

假设你并不知道Flux架构模式，那就从头讲起。

随着单页应用越来越复杂，应用中的状态也变的错综复杂。对于这些状态的管理也变的非常困难，在这过程中也出现了很多的解决方案，比如前端MVC架构。但是在MVC框架中并不能严格遵守一个Model对应一个View，一个View会去修改几个Model，一个Model的变化也会引起其它Model的变化，所以在复杂中应用中这种模式状态的管理依然很混乱。Redux就是用来管理应用状态的一个库，它把应用中的状态集中在一个地方，只能通过特定的方式，定义好的逻辑来修改状态。这样状态的变化就变的可预测且可追溯。

### 三大原则
想要理解Redux必须要知道Redux设计和使用的三大原则。
##### 单一数据源
Redux把整个应用的状态都保存在应用唯一的store中。不像MVC可能会存在很多的Model以及Model之间的互相引用修改，所有状态的变化都必须在这个store中发生，所有的View有同一个数据源。数据的追溯和调试也变得简单。
##### State是只读的
Redux不允许直接修改store中的数据，Redux的store也没有提供setter这样的接口。修改store的唯一方式就是分发一个action，action描述了要进行什么样的修改，但是并不负责执行修改。这些action会被集中响应，完成状态的修改。
##### 使用纯函数来执行修改
上面说描述修改的action会被reducer响应，状态的修改就是被这些被称为reducer的纯函数执行的。
reducer接收先前的state和分发的action，返回新的state。reducer可以是一个或多个用来操作状态的不同部分，最后再合并在一起。

reducer是纯函数，这样就确保了同样输入一定会得到同样的输出。这样使得状态的修改变的简单纯粹，可测试。

### Redux的基本概念

Redux由Action、Reducer和Store这三个部分组成。

##### Action
前面我们已经提到过action。在Redux文档给出的定义是:
**Action**是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。Action本质上是一个普通的javascript对象。在定义action的时候一般都会遵守FSA（Flux Standard Action）标准。action中必须包含一个`type`字段来表示要执行的动作类型，`type`必须是一个字符串类型的值。

一个action就像下面这样
```
{
    type: 'TODO_ADD',
    payload: 1,
}
```
通常情况下，我们并不会直接定义一个action，而是通过action创建函数生成action。记住，action是一个javascript对象，在实际项目中见得最多的是action创建函数。

下面是一个action创建函数的示例，它返回了一个action。

```
function addTodo(payload) {
  return {
    type: ADD_TODO,
    payload
  }
}
```
用action创建函数生成action减少了代码的重复，使得更容易移植和测试。

##### Reducer
Reducer的作用响应action修改store中的数据，reducer也是修改store的唯一方法。

reducer是一个接收前一个state和当前action并且返回一个新的state**纯函数**。用一行代码表示：
```
(preState, action) => newState
```
记住，要永远保持reducer的纯净，不要引入副作用。只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。

下面我们就编写一个可以响应action的reducer。
```
function todoApp(state = initialState, action) {
  switch (action.type) {
    case 'TODO_SOMETHING':
      return Object.assign({}, state, {
        data: action.payload
      })
    default:
      return state
  }
}
```
编写reducer时，不要修改旧的state，一定要复制一个新的state进行操作，最后返回一个全新的state。reducer必须有返回，默认返回旧的state。

当应用比较复杂的时候，这个reducer就会变的非常大，这不是我们想要的。一个应用中可以把reducer拆分成若干个小的碎片，最后合并在一起，Redux为我们提供了一个`combineReducers()`方法来合并reducer。
```
const reducer = combineReducers({
    reducer1,
    reducer2,
})
```
##### Store
Redux并没有提供类似Flux中dispatcher这样的概念，所以
Store除了用来维持应用的state，store也是联系action和reducer的枢纽。store并不等同于应用的state，更像是一个容器，在维持了应用的state的同时提供了几个与其它部分链接的接口。

- `getState`方法获取state；
- `dispatch`方法更新state；
- `subscribe`方法注册监听器；
- `unsubscribe`方法取消监听。

Redux提供了一个创建store的方法`createStore`，把之前定义好的reducer传给这个store创建函数，Redux会根据reducer生成应用的state，`dispatch`分发action，reducer就会响应收到的action修改state。

虽然看起来我们把reducer拆分成了多个碎片，但是在传入`createStore`又合并成了一个，实际上每个被拆分的reducer都会收到这个action，不过只有其中的一个会响应。在编写reducer的时候要注意，一种类型的action只能被一个reducer响应。

当state修改完成后，我们要通知关心应用state的部分，比如说视图层(View)。`subscribe`方法就是用来给state注册一个监听器，监听state的更新作出反应。

上面的`subscribe`方法会返回一个取消监听的函数，调用这个函数可以取消监听，`unsubscribe`并不是store提供的确定的API，只是对`subscribe`返回函数的一般叫法。
```
import { createStore } from 'redux'
import reducers from './reducers'
import { todo } from './actions'

// 创建store
let store = createStore(reducers)

// 给state添加一个监听器
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// 发起action
store.dispatch(todo('do something'))

// 停止监听 state 更新
unsubscribe();

```
`createStore`还接收第二个可选参数作为state的初始值。

### Redux工作流程
在了解了Redux的各个组成部分后，大概就已经清楚了Redux的数据工作流程了。



第一步，调用`store.dispatch(action)`发出一个执行动作。

第二步，store调用我们在创建store时传入的reducer，并把当前store中state和上一步中的action传入reducer函数中，reducer根据`action.type`的值决定修改state的逻辑。

第三步，reducer把所有的子reducer计算输出的state合并到一个state树中。

第四步，store保存reducer返回的新的state树，订阅了state的监听器被调用，如果监听state的视图层，就是视图层的重新渲染。

#### 绑定React
Redux目前最广泛的用法就是把React作为视图层与Redux进行绑定，但是Redux与React之间并没有必然的关系。Redux官方提供了一个react-redux库，帮助我们对react和Redux绑定。如同Redux一样简洁。一个React组件`<Provider/>`，一个方法`connect()`。关于它们，只需要知道的是`<Provider/>`接收一个store作为props，它是Redux应用的顶层组件。实际上`<Provider/>`就是一个很简单的React组件，在构造函数中获取props中的store，并把获取到的store挂在在当前实例上，然后通过`getChildContext`方法把store传递下去。而 connect() 提供了在整个 React应用的任意组件中获取 store 中数据的功能。connect()方法接收四个参数，我们用的最是前两个参数，它们作用是从store中的state树中找到组件中需要的数据和需要修改state树的方法当作props传给将要包装的组件。具体使用请查看文档，本篇不做介绍。

### 总结
上面的只是对Redux的一个基本介绍和机制，并不能作为Redux的实际使用教程，因为还有一些非常重要但并不是Redux必须的用法没有涉及到，比如说中间件，还有一些其它的API等。如果要用好Redux，就不得不对中间件有深入的了解。