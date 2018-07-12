React自己实现了一套事件系统，在底层填平了不同浏览器之间的兼容问题，提供了一套与原生DOM事件非常类似的统一接口，使得我们在编码中可以专注于业务而不用为了繁琐的跨浏览器的问题而头疼。于此同时，也尽可能的降低了因为开发者水平不同而引起的性能问题。

#### 基本用法

React中的事件使用驼峰命名法，不同于DOM事件的全小写命名。在JSX中的写法如下：
```
// ...

function handleClick(e) {
    // 
}
// ...
<div onClick={handleClick}>
```
#### 合成事件对象
事件处理函数会得到一个合成事件对象`SyntheticEvent`作为参数，但是这个对象并不是原生的事件对象，而是React根据W3C规范定的一个合成事件对象，在这里我们不用关心浏览器的兼容问题。底层浏览器事件对象被封装在了`nativeEvent`属性里，以便开发人员在有需要的时候获取。

需要注意的是，合成事件对象`SyntheticEvent`是不能用在异步的代码中的。React事件系统中维持了一个事件池，传给事件处理程序的事件对象就是由原生对象和事件池中对应的事件对象合并而成的，一旦事件处理程序执行完毕，这个处理程序中的事件对象就会被回收。

如果有需要可以将这个事件对象中的某个属性或者整个对象保存在其它地方，以便在事件处理程序结束以后使用。
```
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);

  // 不能工作。 this.state.click 事件只包含空值。
  this.setState({clickEvent: event});

  // 您仍然可以导出事件属性。
  this.setState({eventType: event.type});
}
```

在事件系统中React重写的事件流的实现，要阻止React合成事件的冒泡需要显式的调用`e.stopPropagation()`或者`e.preventDefault()`。这两个方法不能阻止原生事件的冒泡。

#### 关于事件处理程序中的this值

在DOM原生事件处理程序中，this值指向的是事件处理对象，但是在React中，我们通常希望`this`值指向当前的组件实例，这种情况下我们可以在构造函数中为事件处理程序绑定`this`值：
```
class A extends Component {
    constructor(props) {
        super(props);
        this.something = 'hahaha'
        this.handleClick = this.handleClick.bind(this);
    }
    
    handleClick(e) {
        console.log(this.something)  // hahaha
    }
    
    render() {
        return (
        <div onClick={this.handleClick}>点我</div>)
    }
}
```
除此之外用箭头函数绑定`this`值也是非常不错的选择，强烈推荐：
```
class A extends Component {
    constructor(props) {
        super(props);
        this.something = 'hahaha'
    }
    
    handleClick = (e) => {
        console.log(this.something)  // hahaha
    }
    
    render() {
        return (
        <div onClick={this.handleClick}>点我</div>)
    }
}
```


#### 深入原理

React利用了事件委托将所有的的事件都注册在`document`这个根节点上，这样就大大的减少了原生事件，也就意味这更少的内存开销，同时React的事件系统利用我们前面提到的事件池管理合成事件，在需要的时候创建，在执行完成之后回收，减少事件对象的内存占用。

##### 1、事件的注册

在组件注册注册或更新的过程中，React会检查注入到这个组件上的所有的属性(props)名，看是否是一个普通的属性还是一个事件注册。如果是一个事件注册，接下来React会做两件事

 - 首先，给`document`注册对应的原生事件。这里在`document`上注册的原生事件并不是真正的事件处理程序，只是一个叫做`dispatchEvent`的方法，这个方法中存储了原生事件对应的React中的`topLevelType`作为参数，以便事件执行时的调用。
 - 然后，存储事件处理程序会。真正的事件处理程序会通过调用`putListeners(inst, propKey)`方法统一存储在`listenerBank`中，格式是`listenerBank[合成事件类型名称][组件实例id] = litener`。如果是在组件更新中，则还会调用``deleteListener(inst, propKey)`删除已经不存在事件。
 
##### 2、事件的执行

当原生事件被触发时，事件最终会冒泡到`document`节点上（如果原生事件的冒泡没有被阻止）。在注册阶段注册在`document`节点上的`dispatchEvent`事件就会被调用，此时原生事件对象会被作为第二个参数传入。
```
dispatchEvent: function (topLevelType, nativeEvent) {
    // ...
    // 取回注册事件的时候存储的回调方法
    var bookKeeping = TopLevelCallbackBookKeeping.getPooled(topLevelType, nativeEvent);
    try {
      // 放入批处理队列中
      ReactUpdates.batchedUpdates(handleTopLevelImpl, bookKeeping);
    } finally {
      TopLevelCallbackBookKeeping.release(bookKeeping);
    }
}
```
在`dispatchEvent`方法的最后会回收这个对象，这也就是我们上面提到过合成事件对象不可用在异步代码中的原因。

React在拿到原生事件之后就可以在组件树中获取到组件实例。然后利用这些参数构造出一个合成事件。有了最初的事件处理回调以及合成事件之后，就具备了本次事件执行的所有条件了。但是实际上并非这么简单，之前我们说过React实现了事件流冒泡机制。React冒泡的实现就是通过遍历注册事件程序的那个组件的上级组件，然后执行注册在每个祖先组件上的此类原生事件。
```
function handleTopLevelImpl(bookKeeping) {
  // 获取原生DOM对应的React 组件
  var nativeEventTarget = getEventTarget(bookKeeping.nativeEvent);
  var targetInst = ReactDOMComponentTree.getClosestInstanceFromNode(nativeEventTarget);

  // 遍历组件树，将targetInst的祖先组件放在bookKeeping.ancestors这个数组中
  var ancestor = targetInst;
  do {
    bookKeeping.ancestors.push(ancestor);
    ancestor = ancestor && findParent(ancestor);
  } while (ancestor);

  // 遍历父级组件数组，执行事件处理程序
  for (var i = 0; i < bookKeeping.ancestors.length; i++) {
    targetInst = bookKeeping.ancestors[i];
    ReactEventListener._handleTopLevel(bookKeeping.topLevelType, targetInst, bookKeeping.nativeEvent, getEventTarget(bookKeeping.nativeEvent));
  }
}
```
在这里我们并没有看到相关的阻止冒泡的代码，实际上这里并没有执行处理程序，而是放在了一个批量处理队列中，而阻止冒泡的代码正是在批量执行中实现的。


最后合成事件对象的获取，释放等一系列操作都是在`_handleTopLevel`方法中完成的，这个方法是事件程序执行的关键方法。至此就完成了React事件系统的大部分流程。

最后，当组件卸载的时候，React会调用`deleteAllListener`删除所有的该组件注册的事件程序。
