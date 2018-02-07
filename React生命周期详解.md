生命周期，如果从普遍意义上来理解就是一个事物的阶段性变化及其规律。同样我们也可以用这个定义来理解React组件的生命周期。所以React组件的生命周期就是一个组件的阶段变化和规律。生命周期也是React的核心概念之一，所以清楚理解其中的各个阶段、规律对于学习React来说是绕不开的。

React组件的生命周期可以分为挂载阶段，渲染和卸载三个阶段。在不同的阶段生命周期提供了一些不同的方法，这些方法被称为生命周期钩子。
- #### 组件的挂载（mountComponent）
挂载是组件生命周期的第一个阶段，这个过程主要做的就是组件的初始化。一个组件挂载生命周期大概可以用下面的代码来表示：
```
import React from 'react';

class MyComponent extends React.Component {

  static defaultProps = {
    // 初始 props
  };
  
  constructor(props) {
    super(props);
    this.state = {
        // 初始状态 
    }
  }

  componentWillMount() {
    // 即将挂载
  }

  componentDidMount() {
    // 挂载完成
  }
  
  render() {
    // 渲染
    return (
      <div>
        <h1>Hello, world!</h1>
      </div>
    );
  }
}
```
组件首次挂载时，会按顺序执行`getDefaultProps`、`getInitialState`、`componentWillMount`、`render`和`componentDidMount`这几个生命周期方法。

`getDefaultProps`方法是在组件的构造函数中执行的，所以也是生命周期中最早执行的。同样也是因为在构造函数中执行，所以它只会在执行构造函数的时候执行一次，在以后的将都不会执行。

然后组件通过mountComponent开始组件的初始化工作，诸如初始化props、content等参数，执行`getInitialState`获取初始化状态，初始化更新队列和更新状态，执行组件的初始化挂载。

如果组件中存在`componentWillMount`方法。接着就是组件的渲染`render`。`componentDidMount`是在组件渲染完成之后执行的。

如果在当前的组件中嵌套了其它的组件，这个时候的这两个组件各自的生命周期是一个什么样的关系。其实React组件在渲染的时候是通过递归来完成的，父组件的渲染早于子组件的渲染，并且只有所有的子组件渲染完成后，父组件的渲染才算完成，也就是说父组件的`componentWillMount`执行比子组件的`componentWillMount`早，`componentDidMount`比子组件晚。用一张图来表示：(生命周期流程图)

![image](https://github.com/Hiker9527/js/blob/master/static/mount-component.png)

- #### 组件存续期间的更新（updateComponent）
当组件初始化挂载完成之后，就由updateComponent负责管理生命周期中的`componentWillReceiveProps`、`shouldComponentUpdate`、`componentWillUpdate`、`componentWillUpdate`、`render`以及`componentDidUpdate`这几个生命周期钩子方法。

```
class MyComponent extends React.Component {

  ...
  componentWillReceiveProps() {
      // 接收属性
  }
  
  shouldComponentUpdate() {
      // 组件是否继续更新
  }
  
  componentWillUpdate() {
    // 即将更新
  }

  componentDidUpdate() {
    // 挂载完成
  }
  
  render() {
    // 渲染
    return (
      <div>
        <h1>Hello, world!</h1>
      </div>
    );
  }
}
```
当组件发生变化的时候，就要通过`updateComponent`来更新组件了。在这个阶段的生命周期方法中，`componentWillReceiveProps`首先被调用，React会把最新的`props`传给这个方法。

调用`shouldComponentUpdate`可以决定是否要继续更新组件。如果返回`true`就继续更新，否则将停止本次更新。因为`updateCompenent`也是递归更新所有子组件的，可以子组件中利用这个函数来阻止那些没有必要更新的，提升应用的性能。

如果没有提供`shouldComponentUpdate`方法或者这个方法返回了`true`，updateComponent就会继续往下执行，来到`componentWillUpdate`,接着更新视图，完成组件的更新后执行该阶段的最后一个方法`componentDidUpdate`。

不要在`shouldComponentUpdate`和`componentWillUpdate`方法中调用`setState`，这样会导致循环调用，浏览器内存耗尽奔溃。

![image](https://github.com/Hiker9527/js/blob/master/static/update-component.png)
- #### 组件卸载（unmoutComponent）
卸载阶段就只提供了一个方法`componentWillUnmount`。如果提供了这个方法，就在这个方法中重置相关参数、更新队列之类的。不过在大多数的场景下这些操作已经不是很重要了。


- #### 函数式组件
创建组件的方式除了用React的Component类外，还有一种就是在一个纯函数中返回要渲染的元素，用这种方式创建的组件，也叫无状态组件，有没有继承Component类，所以这种组件并没有生命周期，只是简单的接收props然后渲染一个DOM。这种组件的的比用构造函数创建的组件又更高的效率，在合适的场景下应该尽量使用这种组件。

- #### 在生命周期中使用`setState`的时机

由于`setState`的工作机制，使得在有些生命周期方法中是禁止调用`setState`的，有的方法方法中调用了也并不会马上生效。
用一张图来总结这个关系:

![image](https://github.com/Hiker9527/js/blob/master/static/whole-life.png)

图中打了叉的是禁止调用`setState`的，没有表明的表示可以调用，但是不会马上更新状态。这跟React的`setState`的实现机制有关，在这些生命追周期中调用`setState`，状态可能表现为异步更新，并且不会再次出发re-render。如果想彻底搞清楚`setState`为什么会为异步，还要深入理解`setState`的原理，这里不再分析。