# 组件和属性
组件可以让你把UI分割为独立的，可复用的零件，每个零件都可以认为是孤立的。

从概念上讲，组件更像是javascript函数，它能接收任意的输入（叫做"props"）并且返回React元素来描述如何在屏幕上进行展示

### 函数和类组件
创建一个组件最简单的方法是写一个JavaScript函数:

    function Welcome(props) {
        return <h1>Hello,{props.name}</h1>
    }
这个函数就是一个有效的React组件，它接收单一的包含数据的"props"对象参数，返回一个React元素。这样的组件被称为函数式的，因为它们就是字面量的JavaScript函数

也可以使用ES6 class定义组件

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
从React视图的观点看上面的两个组件都是一样的

类还有一些额外的特性，在下一节中会讨论。那时我们会用函数组件的简洁性

### 组件渲染
之前，我们遇到的React元素只表示DOM元素
```
const element = <div />;
```
但是，元素还可以表示自定义的组件
```
const element = <Welcome name="Sara" />;
```
当React看到一个元素表示自定义的组件的时候，就会把JSX属性作为一个单一的对象传递给这个组件，这个对象我们称为"props".

举个例子，下面的代码把"Hello,Sara"渲染到页面上:
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
回顾一下刚才发生了什么:

1. 在`<Welcome name="Sara" />`元素上调用了`ReactDOM.render()`
2. React调用`Welcome`组件并把`{name: 'Sara'}`作为组件的属性
3. `Welcome`组件返回`<h1>Hello, Sara</h1>`元素
4. React DOM成功的更新DOM来匹配`<h1>Hello, Sara</h1>`

>**注意**
>
>组件名的首字母必须大写
>例如: `<div />`表示一个DOM标签，但是`<Welcome />`表示一个组件，并且要求`Welcome`必须在范围内

### 组合组件
在组件的输出中可以加入其他组件。这使我们在任何级别的细节使用相同的组件。一个button,一个form,一个dialog一个屏幕:在React apps中，这些通常都用组件来表示。

请看下面的示例，创建`App`组件时多次渲染了`Welcome`这个组件:
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

典型的做法是，新的React应用只会在最顶层有一个`App`组件.但是如果是要把React整合到已有的app中，最好是从最底层开始，用一些小的组件，比如`Btton`,逐步到最上面的view层。

> **注意**
>
> 组件只能返回一个根元素，这就是为什么要把所有的<Welcome>元素放到`<div>`中
> 

## 提取组件
不要害怕把组件分割成小的组件

例如，考虑这个`Comment`组件
```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

这个组件接收`author`(对象),`text`(字符串),`date`(日期)作为属性，表示一个社交媒体网站的评论。

由于嵌套的原因，这个组件很难更改，而且也很难复用个性化的部分。下面我们要从这个组件中提取几个组件出来

首先提取`Avatar`:
```
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```
`Avatar`并不需要知道会被渲染到`Comment`中，这就是为什么我们会给它的属性一个更一般化的名字:`user`而不是`author`

给属性命名的时候，我们推荐从组件本身的角度出发而不是要被用到地方

现在我们可以稍微简化一下`Comment`
```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
下面，我们提取`UserInfo`组件，在用户名旁边呈现`Avatar`
```
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```
这样我们就能进一步简化`Comment`

```
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
最初提取组件看起来是一个繁琐的工作，但是这样的话就可以大型应用中拥有一套可以重复使用的组件。一个好的经验规则是，如果UI中的某一部分被重复使用了很多遍，或者这个组件本身就很复杂，那么这部分就应该作为一个可复用的组件.

### 属性是只读的

无论你把一个组件定义为函数还function还是class，它的属性永远就不能被修改。考虑这个`sum`函数
```
function sum(a, b) {
  return a + b;
}
```
这样函数被称为"纯"函数，因为这个函数不会修改输入，同样的输入永远返回同样的结果。

作为对比，这个函数就不是纯函数，因为它修改了输入:
```
function withdraw(account, amount) {
  account.total -= amount;
}
```
React相当灵活同时它也有一条非常严格的规则:

**所有的 React 组件 必须表现的像纯函数，并与其属性（ props ）关联。**

当然，应用的UI是动态的而且一直在变化。下一节我们会介绍一个新的概念"state"。State允许React组件改变它们的输出来响应用户的操作、网络请求和其它的任何操作，并不会违反这条规则。







