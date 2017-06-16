> 在使用redux-observable之前，需要先了解RxJS v5的Observable。如果你以前没有 接触过RxJS的响应式编程的话，最好先去阅读http://reactivex.io/rxjs/。redux-observable只是Observable在redux中的一种包装形式，本质上还是Observable。

redux-observable是解决reudx异步数据流的众多方式中的一种，如果你接触过redux的异步处理，那么你很可能知道redux-thunk，它是redux的另外一种异步实现方法，它的原理是在给action传入一个有副作用的函数。redux-thunk相对来说比较容易理解也好上手，但是这也意味着不够强大。redux-observable是基于RxJS的可观察对象，如果你了解Observable，你就会知道redux-observable非常强大，可以实现很复杂副作用也正是它出名的原因。如果你喜欢RxJS，你可以把它用在任何地方。

## Epics
Epics是redux-observable核心基元。它是一个函数，redux就是靠Epics来实现异步数据流的处理的。在这个函数中传入一个action流，然后返回一个action流。
```
function (action$: Observable<Action>, store: Store): Observable<Action>;
```
上面的函数中我们传入了一个`action$`，用'$'符结尾是用来标识这个变量引用了一个流。你发射的这个action将会通过正常的`store.dispatch()`方法立即触发。其实redux-observable实际上就是`epic(action$, store).subscribe(store.dispatch)`。

Epics与正常的redux触发渠道并肩运行，而且是在被reducer已经接收到action之后，所以你没办法"吞掉(swallow)"一个输入的action。action总是在你的Epics接收到它们之前先通过你的reducer运算。

如果你这么干，你就会形成一个死循环：
```
// DO NOT DO THIS
const actionEpic = (action$) => action$; // creates infinite loop
```


## 基本示例
这是一个简单的例子：
```
const pingEpic = action$ =>
  action$.filter(action => action.type === 'PING')
    .mapTo({ type: 'PONG' });

// later...
dispatch({ type: 'PING' });
```

我们触发了`PING`这个action，在action$流中过滤出相应的action，然后用`mapTo`这个操作符映射到`{type: 'PONG'}`这个action上去。Epic函数会对最后发射的这个action自动调用`store.dispatch`，然后触发这个action。

所以，这段代码实现的功能与下面这两行是一样的
```
dispatch({type: 'PING'});
dispatch({type: 'PONG'});
```
这是一个同步的触发，接下来我们实现一个异步请求事件：
```
import { ajax } from 'rxjs/observable/dom/ajax';

// action creators
const fetchUser = username => ({ type: FETCH_USER, payload: username });
const fetchUserFulfilled = payload => ({ type: FETCH_USER_FULFILLED, payload });

// epic
const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
      ajax.getJSON(`https://api.github.com/users/${action.payload}`)
        .map(response => fetchUserFulfilled(response))
    );

// later...
dispatch(fetchUser('torvalds'));
```
> 注意在这里我们用到了`ofType()`这个方法来代替`fiter()`方法，由于`filter`用的比较多，所以redux-observable就自定义了这么一个方法。

上面的代码中我们手动触发了`fetchUser()`这个action。产生的流会执行后面的AJAX请求。当AJAX请求完成返回后，我们把返回的响应传入后面的`FETCH_USER_FULFILLED`action。
> 记住，==Epics接收一个**action**流，并且返回一个**action**流==。如果你对此感到疑惑，那么你应该先去看看RxJS

然后我们用AJAX的响应数据来更新store:
```
const users = (state = {}, action) => {
  switch (action.type) {
    case FETCH_USER_FULFILLED:
      return {
        ...state,
        // `login` is the username
        [action.payload.login]: action.payload
      };

    default:
      return state;
  }
};
```

手动触发`fetchUser('torvalds')`这个action，过滤出相应的action,使用mergeMap操作符，内部是这样一个流，调用ajax流成功从接口拉取到数据后，把数据map到另外一个action流上，这个action流会把刚才接收到的数据一起发射出去，并触发相应的action，此时我们的reducer就会收到数据，更新store。外部的mergeMap操作符就把内部最终发射的带有数据的action作为输出流的发射值，触发被发射的这个action。

上面就是redux-observable处理Redux异步事件的基本原理和流程。

## 接入Store的state
Epics函数还能接受第二个参数，一个轻量级的Redux store。
```
type LightStore = { getState: Function, dispatch: Function };

function (action$: ActionsObservable<Action>, store: LightStore ): ActionsObservable<Action>;
```
这个轻量级的store只是包含了store的两个方法，并不是整个store。在应用中我们需要调用`getState`来获取当前的state。
```
const INCREMENT = 'INCREMENT';
const INCREMENT_IF_ODD = 'INCREMENT_IF_ODD';

const increment = () => ({ type: INCREMENT });
const incrementIfOdd = () => ({ type: INCREMENT_IF_ODD });

const incrementIfOddEpic = (action$, store) =>
  action$.ofType(INCREMENT_IF_ODD)
    .filter(() => store.getState().counter % 2 === 1)
    .map(() => increment());

// later...
dispatch(incrementIfOdd());
```
## 整合到Redux中


前面创建的Epics函数中我们都是通过手动触发action，这显然不是我们希望看到的样子。下面我们会把Epics函数放在redux-obervable中间件中，这样Eipcs函数就可以自动监听action了。

就像redux只需要一个根reducer一样，redux-observable也只能有一个根Epics，所以我们先把所有的Eipcs函数合并成一个。

redux-observable提供了一个工具`combineEpics()`，它能很方便的把多个的Epics合并成一个Epics。这个`combineEpics`实际上就是RxJS的`merge`操作符的一个包装。这样我们就能在各自的模块中写自己的Epics函数，最后合并成一个，跟reducer一样。
```
//redux/modules/root.js
import { combineEpics } from 'redux-observable';
import { combineReducers } from 'redux';
import ping, { pingEpic } from './ping';
import users, { fetchUserEpic } from './users';

export const rootEpic = combineEpics(
  pingEpic,
  fetchUserEpic
);

export const rootReducer = combineReducers({
  ping,
  users
});
```
接下来配置我们的Store：
```
//redux/configureStore.js
import { createStore, applyMiddleware } from 'redux';
import { createEpicMiddleware } from 'redux-observable';
import { rootEpic, rootReducer } from './modules/root';

const epicMiddleware = createEpicMiddleware(rootEpic);

export default function configureStore() {
  const store = createStore(
    rootReducer,
    applyMiddleware(epicMiddleware)
  );


  return store;
}
```
如果是在开发环境中，我们还需要Redux DevTools
```
import { compose } from 'redux'; // and your other imports from before
const epicMiddleware = createEpicMiddleware(pingEpic);

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

const store = createStore(pingReducer,
  composeEnhancers(
    applyMiddleware(epicMiddleware)
  )
);
```

更多高级功能请移步[redux-observable官方文档](https://redux-observable.js.org/)

（本文章为本人学习笔记，由于水平有限所以有许多不严谨的地方。若有人看到这篇文章，有疑惑的地方请参考官方文档）
