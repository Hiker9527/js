Flux是什么？Redux与Flux是什么关系？

Flux是由facebook工程师提出来的，用来解决React状态管理的一种方案。由于React是严格的单向数据流，但是在复杂的应用中跨组件的数据交流是无法避免的，flux的出现就是针对数据单向流中数据共享的问题的。

正如其名，Flux的核心思想就是数据和逻辑的永远单向流动。

![image](https://github.com/Hiker9527/js/static/flux-overview.png)

在Flux应用中，数据Action到dispatcher，再到store，最终到view的路线永远是单向不可逆的。在Flux中，所有的数据都保存在store中，修改store的唯一方法是通过dispatcher分发action，保证了store数据在应用中的确定性。

#### Flux的概念
一个flux应用由三个部分组成：<br>
1、dispatcher负责事件分发<br>
2、store负责保存数据，同时响应事件并更新数据<br>
3、view订阅store中的数据，并使用这些数据去渲染页面

##### dispatcher与action

dispatcher是Flux的核心的概念。Flux应用正式通过dispatcher来分发应用中的事件的。dispatcher有两个最核心的方法`regiester`和`dispatch`。

`regiester`方法用来注册一个监听器，`dispatch`用来分发一个action。

action就是一个简单的javascript对象，用来描述一个事件以及所要传递的数据。一般包含type和payload等字段字段。其中type为必须字段，表示该事件的类型。payload一般表示此次事件的荷载，也就是数据。

关于如何定义一个action，有一个称为FSA(Flux Standard Action)的规范，在定义action时请遵守此规范。

##### store
store负责保存Flux应用中的数据，并定义修改数据的逻辑，同时调用dispatcher的register将自己注册为一个监听器。这样一旦我们使用dispatch分发一个action的时候，store注册的监听器就会被调用，同时得到action作为参数。

store注册的监听器拿到参数action后，根据action的type值判断是否响应这个action，以及用哪个逻辑修改store中的数据数据。修改后的store会触发一个更新事件，通知view重新渲染视图。

##### controller-view
我们前面讲的Flux三大组成部分并没有包括controller-view。controller-view不是一个单独的部分，它是一个具有特殊功能的view。
store中保存了应用的数据，而这些数据最终是要被view所用，controller-view的作用就是将store绑定到view中。

一般情况下应用最顶层的view就是controller-view。。controller-view不会包含业务逻辑，它的主要作用就是store的绑定和数据的传递。

controller-view通过store暴露的getter方法获取到store中的数据，然后用这些数据设置自己的state，最后在render中以props的方式传递给下层view，这样整个应用中的任何地方就都可以拿到store中的数据。

前面的store中说到store在响应action更新store后会触发一个更新，这个更新就是在controller-view中监听的。controller-view监听到store更新后调用setState更新组件的状态，触发re-render，并把新的store数据传递给下层。

##### view
到这里我们就完成了数据从store到view的流动了，用view把store中的信息展示给用户。view还有一个作用就是接收用户的输入。当用户的某个输入需要修改store中的数据时，我们前面说过修改store的唯一办法就是调用dispatcher的dispatch方法分发一个action，由store去响应这个action。所以如果要在view中修改store，那么就要分发一个相应的action。

#### dispatcher的实现
facebook对于Flux中的dispatcher的实现其实很简单。最主要的两个方法就是`register`和`dispatch`。

下面我就简单的分析一下它的源码的实现机制：
```
var _prefix = 'ID_';

class Dispatcher {
  constructor() {
    this._callbacks = {};
    this._isDispatching = false;
    this._isHandled = {};
    this._isPending = {};
    this._lastID = 1;
  }
  
  // ...
  // 监听器注册方法，把callback存在实例的_callbacks中
  // 并且返回一个id以便取消监听的时候使用
  register(callback) {
      var id = _prefix + this._lastID++;
      this._callbacks[id] = callback;
      return id
  }
  dispatch(payload) {
    // 这里是log输出，不用管
    invariant(
      !this._isDispatching,
      'Dispatch.dispatch(...): Cannot dispatch in the middle of a dispatch.'
    );
    // 调用_startDispatching方法，将dispatching状态置为true，
    // 同时把接收到的payload参数暂存在this._pendingPayload中
    this._startDispatching(payload);
    
    try {
        // 遍历所有注册的监听器，如果不是pending状态，就执行监听函数
        for (var id in this._callbacks) {
            if (this._isPending[id]) {
                continue;
            }
            this._invokeCallback(id);
        }
    } finally {
        // 重置状态
        this._stopDispatching();
    }
  }
  // 执行器
  _invokeCallback(id) {
      this._isPending[id] = true;
      this._callbacks[id](this._pendingPayload);
      this._isHandled[id] = true;
  }
  
}
```

以上就是dispatcher两个核心方法的源码，简洁但是依然很强大。

#### 总结

至此，我们就把Flux的完整流程梳理完毕了。回到我们最开始的问题，Flux到底是个什么东西，一个框架？一个库？其实都不是，Flux只是一种前端应用的架构模式，一种思想。用很少的代码就实现了一个媲美其它前端MVC的架构。在Flux应用可以使用facebook的dispatcher实现，也可以按照这种模式自己去实现一个dispatcher。Redux就是一个很好的实现了Flux的库，也是目前最流行的一个库。

前面我们在介绍中view默认是基于React来实现的，但是并不限于React，Flux同样也可以结合Vue、Angular来使用。

我们再回想一遍Flux实现过程，它的核心的是一个叫dispatcher的东西，dispatcher的dispatch方法用来分发事件，register方法用来注册一个监听器，定义修改store数据的逻辑。应用中有一个叫store地方存放了数据，只暴露给外界一个getter方法。store调用dispatcher的register方法给自己注册一个监听器，一旦在应用中的某个地方调用了dispatcher的dispatch方法分发了一个action，store中注册的监听器就会响应这个action，修改store中的数据，触发一个更新方法。在controller-view中用store暴露出来的getter方法获取store中的数据，此时store中的数据到了应用中，然后controller-view用某种方式在应用中共享。controller-view中还有一个监听器来响应store更新后触发的更新函数，然后重新获取store中数据共享给应用。当view在响应用户的输入时，就可以用dispatch分发一个action，store就能响应这个事件。当然dispatch并不仅仅是在用户输入的时候调用，也可以是其它任何场景。