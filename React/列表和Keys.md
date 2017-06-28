# 列表和Keys

我们先回顾一下在JavaScript中是如何变换list的

下面给出的代码，我们用`map()`函数取到数组`number`的值然后再乘二.我们把由`map()`返回的新数组赋值给变量`doubled`然后打印出来:
```
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled)
```
上面的代码会在控制台打印出`[2, 4, 6, 8, 10]`.

在React中，把数组变换成元素组成的list几乎是一样的.

## 渲染多个组件
你可以创建一个元素合集然后用花括号`{}`把它们包括在JSX中.

下面，我们用Javascript的`map()`函数循环`numbers`数组.我们为每一项返回一个`<li>`元素.最后，我们把这个元素组成的数组赋值给`listItem`:

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((numbers) => 
    <li>{number}</li>
);
```

我们把整个`listItem`数组放在`<ul>`元素中,然后把它渲染到DOM中:
```
ReactDOM.render(
    <ul>{listItem}<ul>,
    document.getElementById('root')
);
```
这段代码展示了一个从1到5的项目列表.

## 基本组件列表

通常你会把list放在一个组件中.

我们可以把重构上面的例子，把它放在一个接收一个`numbers`数组并输出一个无序列表的组件中.
```
function NumberList(props) {
    const numbers = props.numbers;
    const listItem = numbers.map((number) => 
    <li>{number}</li>
    );
    
    return (
        <ul>{listItem}<ul>
    );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
    <NumberList numbers={numbers} />,
    document.getElementById('root')
)
```
当你运行这段代码的时候，会报一个`a key should be provided for list items`的警告,"Key"是一个在创建元素列表的时候必须包含的特殊 ==字符串属性==.我们将在下一节中讨论它的重要性.

让我们把`key`赋值给`numbers.map()`中的列表项,修复丢失`key`的问题.

```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
## Keys ##

keys帮助React识别哪一项发生了改变，增加,或者移除.Keys应该给数组内部的元素以使元素具有稳定的标识:
```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```
选择key的最好的方法是用一个在它的兄弟中能唯一标识列这个表元素的字符串.大多数情况下你应该用Data中的ID作为key：

```
const todoItems = todos.map((todo) => 
    <li key={todo.id}>
        {todo.text}
    </li>
)
```
当你没有一个稳定的ID用来渲染项目的时候,最后的办法就是使用项目的索引作为key:
```
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```
如果项目能重新排序的话，我们不推荐用索引作为key，这样会变慢。如果你有兴趣的话，可以阅读[深入理解为什么key是必要的](https://facebook.github.io/react/docs/reconciliation.html#recursing-on-children)。

### 用Keys提取组件

keys仅仅在数组上下文中才有意义.

举个例子,如果你提取一个`ListItem`组件，你应该把key放在数组中的`<ListItem />`元素，而不是`ListItem`的根元素`<li>`元素上。

#### Eg: Key的错误用法
```
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong! There is no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Wrong! The key should have been specified here:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
#### Key的正确用法
```
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correct! Key should be specified inside the array.
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
一条好的经验规则是`map()`里面的调用都需要keys。
## 在兄弟中Keys必须是唯一的
keys被用在数组中时，在它的兄弟中必须是唯一的。但是他们不需要全局唯一。当我们生成两个不同的数组时，我们就可以使用同样的keys。

```
function Blog(props) {
    const sidebar = (
        <ul>
            {props.post.map((post) => 
                <li key={post.id}>
                    {post.title}
                </li>
            )}
        </ul>
    );
    const content = props.posts.map((post) => 
        <div key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.content}</p>
        </div>
    );
    return (
        <div>
            {sidebar}
            <hr />
            {content}
        </div>
    );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```
Keys对于React来说就是一个线索，但是它们不会传给你的组件。如果你的组件需要同样的值，那么你就用一个不同的名字显式的当做props传给组件。
```
const content = posts.map((post) => 
    <Post
        key={post.id}
        id={post.id}
        title={post.title}
    />
)
```
上面的例子中，`Post`组件能读到`props.id`,但是读不到`props.key`。

## 在JSX中嵌入map()
上面的例子中我们声明了一个单独的`listItem`变量并且包含在JSX中：
```
function NumberList(props) {
    const numbers = props.numbers;
    const listItems = numbers.map((number) => 
        <ListItem key={number.toString()}
                    value={number} />
    );
    
    return (
        <ul>
            {listItems}
        </ul>
    );
}
```
JSX允许在花括号('{}')中内嵌任何的表达式，所以我们可以内联`map()`的结果：
```
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```
有时候这个结果会是一个清晰的代码，但是这个方式也会被滥用。就像在Javascript中，这取决于你是否觉得把它提取为一个变量可以让代码变得更易读。如果你觉得`map()`的身体嵌套太深的时候，这应该提取变量的时候了。