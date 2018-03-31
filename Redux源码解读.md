之前讲过Redux的工作机制，下面我们将深入Redux源码，解读Redux的实现原理。

### createStore
createStore是Redux是最核心的方法，它提供了Redux实现机制最基本的方法以及生成应用中唯一的store。
createStore.js
```
...
export default function createStore(reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }
  ...
}
```
`createStore`方法接收三个参数：reducer是一个函数，用来生成下一个state树，preloadedState是可选参数，