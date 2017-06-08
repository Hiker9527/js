## 简介
大多数的开发中，API返回的数据通常都是JSON格式的，当我们的应用越来越复杂，不可避免的会存在一些数据的引用。这个时候我们就不得不通过forEach等方法遍历那些嵌套很深的对象或者数组来取出我们需要的数据。这是一件让人非常烦躁而又不得不做的事情。尤其是你的应用中使用Flux或者Redux的时候。

Normalizr是一个很小但是非常强大的工具，他能根据一个模式定义，对JSON进行范式化，把所有数据放到一个对象里，以每条数据里的ID作为该条数据的主键，通过查找ID来实现不同数据的相互引用。避免了大量繁琐重复的细节工作，从而可以更好的关注业务逻辑，使得代码更简洁。

## 基本使用方法
normalizr的使用非常简单，下面就是一个简单使用示例：
```
{
  "id": "123",
  "author": {
    "id": "1",
    "name": "Paul"
  },
  "title": "My awesome blog post",
  "comments": [
    {
      "id": "324",
      "commenter": {
        "id": "2",
        "name": "Nicole"
      }
    }
  ]
}
```
上面的`article`嵌套了两个实体：`uers`和`comments`，下面我们把它格式化为三个实体：
```
import { normalize, schema } from 'normalizr';
 
// Define a users schema 
const user = new schema.Entity('users');
 
// Define your comments schema 
const comment = new schema.Entity('comments', {
  commenter: user
});
 
// Define your article  
const article = new schema.Entity('articles', { 
  author: user,
  comments: [ comment ]
});
 
const normalizedData = normalize(originalData, article);
```
现在,开始的`article`数据变成了这个样子
```
{
  result: "123",
  entities: {
    "articles": { 
      "123": { 
        id: "123",
        author: "1",
        title: "My awesome blog post",
        comments: [ "324" ]
      }
    },
    "users": {
      "1": { "id": "1", "name": "Paul" },
      "2": { "id": "2", "name": "Nicole" }
    },
    "comments": {
      "324": { id: "324", "commenter": "2" }
    }
  }
}
```
首先我们要定义一个模式`schema`，这是nomalizr范式化数据的依据。例子中我们`new schema.Entity('users')`创建了一个`user`模式。这个方法最多可以接收三个参数`Entity(key, definition = {}, options = {})`。

第一个参数是必须的，格式为字符串。这个参数会作为范式化响应体中的一个键名，所有这个类型的实体都会被列在这个键名后面。就像例子中的`user`一样。

第二个参数中是这个实体里面嵌套结构的定义，这个参数是可选的，默认是空对象。上面的例子中，我们定义comments schema的时候，`new schema.Entity('comments', {commenter: user})`中的第二个参数`{commenter: user}`，就是对comments嵌套结构的一个抽象，表明comments里面嵌套了一个`commentor`，它的模式是上面一行定义的`user`。

第三个参数是一个option，有三个配置项：`idAttribute`，`mergeStrategy`和`processStrategy`。用的比较多的是`idAttribute`。我们这里只介绍`idAttribute`项，其余两个有兴趣的可以自己看[文档](https://github.com/paularmstrong/normalizr/blob/master/docs/api.md)。这个属性是用来指定每条数据的唯一标识的，如果没有指定默认使用`id`作为唯一标识。它可以接收一个字符串，也可以接收一个函数。如果是函数，这个函数会按下面的顺序接收三个参数：`value`实体输入的值，`parent`输入数组的父对象，`key`输入数组在父对象中对应的键名。

上面我们创建的`user`schems，用它来范式化的数据只是单条数据。当我们要范式化的数据是一个数组的时候，就要用到另外一个方法：**schema.Array**，它会创建一个用来范式化实体数组的的模式。

这个方法最多接收两个参数，第一个参数是这个数组中所包含的单条数据的模式（schema）。就像我们上面例子中每篇文章下面有很多条评论一样，所以我们在`article`schema中看到`comments: [comment]`，把单个的`comment`schema放在数组中，这其实是`schema.Array(comment)`的简写语法。可以理解为要范式化的数据就是由这样的单个数据组成的数组。这个参数还可以是一个模式到属性值的映射关系。如果你的数组里面不仅只有一种实体类型的时候，就需要有一个映射表来对应数组中每条数据与实体类型的关系。比如下面的例子：
```
const data = [ { id: 1, type: 'admin' }, { id: 2, type: 'user' } ];
const userSchema = new schema.Entity('users');
const adminSchema = new schema.Entity('admins');
const myArray = new schema.Array({
  admins: adminSchema,
  users: userSchema
}, (input, parent, key) => `${input.type}s`);

const normalizedData = normalize(data, myArray);
```
`data`里面有两种类型的用户：`admin`和`user`，所以在范式化后的数据中我们有必要区分这两种类型。首先创建各自的模式(schema)，在`Array`中传入映射关系。`admins`类型的用户数据实体使用`adminSchema`模式，普通用户使用`userSchema`模式。此时`Array`就必须传入第二个参数，这个参数可以是一个字符串或者是函数，表明输入的这个实体的类型。通常会是一个函数，如果是字符串就表示数组中所有的数据都是一个类型。如果传入函数，这个函数按顺序接收三个参数:
+ `value`:单个实体输入的值，就是数组中的单条数据。
+ `parent`:输入数组的父级对象
+ `key`:上面的父级对象的对应的键名。
这个例子中`input`就是对应的`value`,这个函数返回了每条数据中的`${input.type}s`在前面映射表中对应的模式(schema)作为这条数据的schema。最后得到下面的结果：
```
{
  entities: {
    admins: { '1': { id: 1, type: 'admin' } },
    users: { '2': { id: 2, type: 'user' } }
  },
  result: [
    { id: 1, schema: 'admins' },
    { id: 2, schema: 'users' }
  ]
}
```
> 注意，如果第一个参数是映射关系表，必须提供第二个参数。

但是如果返回的数据是一个对象(Object)的时候我们该如何范式化它。这里我们就用到了`schema.Object(definition)`。从字面意思就可以大概理解，创建一个用来范式化对象数据的模式(schema)。它值接收一个参数，这个参数与`schema.Entity(definition)`的第一个参数类似，是要范式化的这个对象的嵌套结构。默认是空对象。这个嵌套关系里面你只需要定义你需要的实体键名就好了，其他的会被复制到范式化的输出中。
```
// Example data response
const data = { users: [ { id: '123', name: 'Beth' } ] };

const user = new schema.Entity('users');
const responseSchema = new schema.Object({ users: new schema.Array(user) });
// or shorthand
const responseSchema = { users: new schema.Array(user) };

const normalizedData = normalize(data, responseSchema);
```
返回的`data`对象中，我们只需要里面的`users`数据，所以我们要先定义`user`模式(schema)，`data`对象里面嵌套了我们要的`users`数组。嵌套关系大概是这样的`{users:  [ user ]}`(`[user]`是`new schema.Array(user)`的简写，个人认为这种简写更直观)。如果返回的对象中还有其他类型的数据，我们可以不用理会。最后的输出是下面这样的：
```
{
  entities: {
    users: { '123': { id_str: '123', name: 'Beth' } }
  },
  result: { users: [ '123' ] }
}
```



上面的用法中都是每一个实体对应一个模式(schema)，当一个实体里面嵌套了两个或以上不同类型的实体的时候，可以使用`schema.Union(definition, schemaAttribute)`方法。第一个参数是输入数组中嵌套关系的映射。第二个参数是每条数据中能表示这条数据要对应的模式类型的键名。可以是字符串或者是函数，作为函数的时候，参数与前面的方法一样。
```
const data = { collaborators: {author: { id: 3, type: 'users', name: 'Mike Persson' },
               reviewer: { id: 2, type: 'groups', name: 'Reviewer Group' }
             }};

const user = new schema.Entity('users');
const group = new schema.Entity('groups');
const unionSchema = new schema.Union({
  user: user,
  group: group
}, 'type');

const normalizedData = normalize(data, { collaborators: unionSchema });
```
`collaborators`嵌套了两种类型的实体`users`和`groups`，我们先要分别定义这两种类型的实体模式，在`Union`中传入映射关系，用每条数据中的`type`键名对应的值来指定这条数据要使用哪种类型的模式(schema)。输出的结果为：
```
{
  entities: {
    users: { '3': { id: 3, type: 'users', name: 'Mike Persson' } },
    groups: { '2': { id: 2, type: 'groups', name: 'Reviewer Group' } }
  },
  result: { collaborators: { author: { id: 3, schema: 'users' }, 
                     reviewer: { id: 2, schema: 'groups' } } 
          }
}
```

最后还有一种方法`Values(definition, schemaAttribute)`。
用来描述值和给定模式(schema)的对应关系。它的两个参数与`Union`基本一致。
```
const data = { firstThing: { id: 1 }, secondThing: { id: 2 } };

const item = new schema.Entity('items');
const valuesSchema = new schema.Values(item);

const normalizedData = normalize(data, valuesSchema);
```
返回的数据是一个对象，里面包含了两个对象`firstThing`和`secondThing`。然后下面先创建了一个`item`的模式(shema)，表示他们下面的项。接着创建`valuesSchema`的时候把`item`这个单个schema当做参数传了进去。输出下面的结果：
```
{
  entities: {
    items: { '1': { id: 1 }, '2': { id: 2 } }
  },
  result: { firstThing: 1, secondThing: 2 }
}
```
`firstThing`的值不再是原来是对象，变成一个指向`items`某个数据的指针。此时的`result`存储的就是一个映射关系。

下面是传入一个映射关系作为第一个参数的例子：

```
const data = {
  '1': { id: 1, type: 'admin' }, 
  '2': { id: 2, type: 'user' }
};

const userSchema = new schema.Entity('users');
const adminSchema = new schema.Entity('admins');
const valuesSchema = new schema.Values({
  admins: adminSchema,
  users: userSchema
}, (input, parent, key) => `${input.type}s`);

const normalizedData = normalize(data, valuesSchema);
```
输出：
```
{
  entities: {
    admins: { '1': { id: 1, type: 'admin' } },
    users: { '2': { id: 2, type: 'user' } }
  },
  result: {
    '1': { id: 1, schema: 'admins' },
    '2': { id: 2, schema: 'users' }
  }
}
```


以上只是对normalizr概念和用法的一个简单介绍。水平有限，还有很多方法高级用法并没有详细讲解，所以建议最好是去研究[官方文档](https://www.npmjs.com/package/normalizr)。

> 参考资料：
>
> 官方文档  https://github.com/paularmstrong/normalizr/blob/master/docs/api.md
>
> [译]JSON数据范式化（normalizr）https://yq.aliyun.com/articles/3168
> 


