### 理解DocumentFragment

`createDocumentFragment`在MDN的解释是:创建一个空白的文档片段(`DocumentFragment`)；所以在理解`createDocumentFragment`之前我们首先要搞清楚什么是`DocuemntFragment`。

##### 什么是`DocumentFragment`

`DocumentFragment`是一个没有父级文件的最小文档对象，被用作一个轻量的文档对象，来暂时存储一些DOM片段。`DocumentFragment`并不是真实的DOM树中的一部分，只是存在于内存中，所以它的变化不会引起DOM树的重绘(reflow)，不会导致性能问题。

最常用的方法是把文档片段(`DocumentFragment`)当作DOM操作的参数(比如`Node.appendChild`或者`Node.insertBefor`)，添加到真实的DOM树中。与其它Node节点不同的是，以这种方法插入到DOM树中的节点是文档片段(`DocumentFragment`)的所有子节点(childNodes)，而并非文档片段本身。

`DocumentFragment`本身也是一种DOM节点，NodeType是11，所以它继承了`Node`的所有属性和方法，并且补充了`ParentNode`接口中的属性和方法。点击查看[具体文档](https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentFragment)。

##### 创建一个DocumentFragment

创建一个空白文档片段(`DocumentFragment`)的方法有两种，一种是使用`createDocumentFragment`方法创建，一种是用构造函数来创建。

创建好空白的文档碎片后，我们就要给它添加文档进去。与元素节点不同的是，文档片段不能使用`innerHTML`添加元素，只能用`appendChild`或者其它类似方法给文档片段内添加文档片段，切记这点区别。
```
const fragment = document.createDocumentFragment();
fragment.appendChild(element);
```

##### 文档片段节点(`DocumentFragment`)与元素节点(`Element`)的区别

事实上就上面提到的文档片段的优点(在内存中，不会引起DOM重绘)使用`createElement`也可以达到一样的效果。创建一个`div`元素节点，然后往里面添加更多的节点，最后一次性插入到DOM中，也只会引起一次DOM重绘。
```
// 创建一个文档片段并插入到DOM中
const fragment = document.createDocumentFragment();
fragment.innerHTML = 'DocumentFragment元素'
...
document.body.appendChild(fragment);

// 创建一个div元素然后插入到DOM中
const element = document.createElement('div');
element.innerHTML = 'element元素';
...
document.body.appendChild(element);
```
如果你试着执行上面的这段代码，你会发现并没有实现你期望的效果。前面说过，`DocumentFragment`不是一个元素节点，所以给它的`innerHTML`属性赋值不会表现的与元素节点一致。正确的使用方法请参考上文。

还有一个重要的区别就是通过`createElement`创建的节点会一直保存在内存中，除非你手动删除引用。而`DocumentFragment`一旦被添加到真实DOM树中之后就会自动销毁。
```
// element
const element = document.createElement('div');
element.innerHTML = 'element元素';
...
document.body.appendChild(element);
// 下面的代码正常执行，element会被再次添加到DOM中
document.body.appendChild(element);

// fragment
const fragment = document.createDocumentFragment();
fragment.innerHTML = 'DocumentFragment元素'
...
document.body.appendChild(fragment);
// 下面会报错，因为fragment的引用已经被销毁
document.body.appendChild(fragment);
```

##### 总结

在开发中至于到底是使用element还是DocumentFragment就需要根据实际场景选择，并不存在谁比谁高级，毕竟合适的才是最好的。