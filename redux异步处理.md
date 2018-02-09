在我接触过的几个React-Web和React-Native项目中几乎每个项目使用的异步处理方式都不同。所以如何在Redux中优雅的处理异步操作是很重要的一个过程。我见过的方案大概有下面四种：

- 组件中直接调用异步方法，完成后手动 dispatch 对应的 action
- redux-thunk
- redux-promise
- redux-saga
- redux-observable

#### 组件中异步回调触发 action
这种方法最简单，非常容易理解，就是回掉函数的处理思路，没有什么好说的。
```
handleFetch() {
    fetch('http://api.xxx.xxx')
        .then(response => response.json())
        .then(json => dispatch(successAction(json))
        .catch(err => dispatch(failedAction(err))
}
```
上面这段代码是容器组件的一个方法，调用这个方法会发起一个异步请求，当请求完成后触发相应的action来更新state。

这样写的好处是不用引入新的文件，所有的代码都在一个文件中，在业务较少的代码中逻辑一目了然。但是一旦在同一个页面中有很多个类似的操作的情况下，我们就需要为每个操作写一个单独方法，最后导致代码非常臃肿，组件之间很难做到重复使用。所以对于这种方式不推荐使用。

#### redux-thunk
这也许是大部分人的第一种异步处理方法，这是写在redux官方文档中的方案，看过文档的都知道这种方法。

我们知道Redux只能处理同步的action，但是我们可以引入`redux-thunk`这个中间来处理异步的action。action creator除了可以返回js对象外也可以返回函数，这个函数既可以是纯函数，也可以是带有副作用的函数，`redux-thunk`知道如何处理函数。
```
// 虽然内部操作不同，你可以像其它 action creator 一样使用它：
// store.dispatch(fetchPosts('reactjs'))
export function fetchPosts(payload) {

  return function (dispatch) {

    // 首次 dispatch：更新应用的 state 来通知
    // API 请求发起了。

    dispatch(requestPosts(payload))

    // thunk middleware 调用的函数可以有返回值，
    // 它会被当作 dispatch 方法的返回值传递。

    // 这个案例中，我们返回一个等待处理的 promise。
    // 这并不是 redux middleware 所必须的，但这对于我们而言很方便。

    return fetch(`http://www.subreddit.com/r/${payload}.json`)
      .then(response => response.json())
      .then(json =>

        // 可以多次 dispatch！
        // 这里，使用 API 请求结果来更新应用的 state。

        dispatch(receivePosts(payload, json))
      )

      // 在实际应用中，还需要
      // 捕获网络请求的异常。
  }
}
```
关于`redux-thunk`的使用在redux的[官方文档](http://www.redux.org.cn/docs/advanced/AsyncActions.html)中有更详细的介绍。

这种方法的好处是把异步操作封装成一个action，我们可以像用同步action的方式来触发异步action。在组件中只需要关心业务逻辑的实现即可。

#### redux-promise
redux-promise相比redux-thunk就要简单的多，可以在直接触发一个promise或者payload 是promise的action。

当redux-promise接受一个promise时，它会dispatch这个promise成功的值，如果这个promise失败则什么都不做。

如果它接受的时一个payload是promise的 Flux Standard Action，会有下面两种结果：
- 触发promise解决后的值，并把`stats`设置为`success`
- 触发promise拒绝后的值，并把`status`设置为`error`
用redux-promise处理一个异步的action只需要短短几行代码
```
export function fetchPosts(id) {
    return {
        type: GET_DATA,
        payload: fetch(`http://www.subreddit.com/r/${payload}.json`)
      .then(response => response.json())
    }
}
```
相比redux-thunk清爽了很多。但是这种做法的后果就是state的所有更新必须是在promise作出反应之后。用户并不能在作出异步操作的第一时间得到反馈。

#### redux-saga
`redux-saga`是一个用与管理Redux副作用的库。saga就像是应用中一个监听action的独立的进程，然后作出响应的处理。saga函数其实就是一个Generator函数，用yield来实现对异步流的控制。

saga提供了很多辅助函数，saga对action的监听以及触发特定的action。比如`takeEvery`用来监听action，然后产生异步流。Effect是saga中的另外一个概念，Effect是saga函数中yield后面的一个javascript对象，这个对象其实就是发送给中间件的一条指令以执行指定的操作。同时Effect让saga函数更易于测试。

利用辅助函数和Effect的组合可以实现很复杂的异步流的控制。

```
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  每次触发`USER_FETCH_REQUESTED` action
  时就执行一次fetchUser
*/
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}
```
在上面这段代码中，saga如果监听到触发了`USER_FETCH_REQUESTED` action，就会执行fetchUser这个Generator函数。执行fetchUser中第一个yield后面的异步请求中，fetchUser函数会暂停，直到这个异步函数执行完毕才回接着执行下一个yield，第二个yield会给middleware发送一个Effect，middleware会据此触发`USER_FETCH_SUCCEEDED`这个action。如果异步请求失败则触发`USER_FETCH_FAILED`。

redux-saga可以利用它提供的API和Generator函数本身的特点实现很强大的异步流控制，同时易于测试性也是redux-saga的一大卖点。而强大能力是建立在更抽象的概念，更复杂的API和更高的运行成本的基础上的。

redux-saga适合用在比较复杂的Redux状态应用中。
在采用redux-saga之前要考虑清除它优点是否值得你付出这样的成本。对于一些异步比较简单的应用，redux-chunk或者redux-promise是更合适的选择。

#### redux-observable

如果你觉得redux-saga已经够强大了，那么redux-observable足以让你觉得惊艳。

redux-observable是rxjs与redux的一个连接，rxjs是js响应式编程的一个非常不错的库，一个可以用在任何地方的库，有非常丰富的API。redux-observable适合大型复杂的状态管理，而且让代码分离度和可维护度更高。在使用redux-observable之前必须先掌握RxJS的基本概念和几个常用的API。

我认为这是我目前为止最优秀的redux异步解决方案。


##### Epics
Epics是redux-observable核心基元。它是一个函数，redux就是靠Epics来实现异步数据流的处理的。在这个函数中传入一个action流，然后返回一个action流。
```
function (action$: Observable<Action>, store: Store): Observable<Action>;
```
当我们触发一个action的时候，所有的Epics函数就会被调用。
Epics与正常的redux触发渠道并肩运行，而且是在被reducer已经接收到action之后。

这样我们就可以在Epics函数中做很多事情，可以是同步的，也可以时异步的。Epics在执行完成后会返回一个action流，返回的action就可以去更新state或者再执行一个Epics函数，只要你愿意，你可以不停的执行下去。
```
// 1. 拉取某用户数据
const fetchUser = username => ({ type: '拉取某用户数据', target: username });
// 2. 拉取完成
const fetchUserDone = data => ({ type: '拉取完成', data});

// 定义一个Epics函数
const fetchEpics = action$ => 
      action$.ofType('拉取某用户数据') // 如果true则进行下一步否则退出
            .mergeMap( action => 
           // 提取action.target并进行ajax请求
           ajax.getJson(`/api/users/${action.target}`)
                .map(function( data ){
              fetchUserDone ( data )// 调用拉取完成函数，返回{ type: '拉取完成', data }
                 })
        );       
```
redux-observable的高明之处在于它把所有的异步操作都提取到了Epics函数中，而Epics函数是action-in，action-out，这样我们就可以对Epics函数采用链式操作。分离了异步操作后的action就都是Flux Standard Action，在复杂的应用中合理的提取对代码的复用和维护是很有必要的。

然后redux-observable的优秀也使得它掌握要比其他方案更加困难。redux-observable的核心是RxJS，如果之前对RxJS不了解的话必须花一定的时间先掌握响应是编程的基本概念和一些API。
