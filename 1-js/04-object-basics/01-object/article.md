
# 对象

我们从 <info:types> 章节知道，在 JavaScript 中一共有 7 种类型，其中 6 种被称为“基本类型”，因为他们的值中只包含一种东西（可能是一个字符串或者一个数值之类）。

相对来说，对象被用来存储多种数据和复杂实体的集合。在 JavaScript 中， 对象的概念渗透到几乎语言里的各个方面。所以在深入其他东西之前我们必须先理解它。

[cut]

一个对象可以通过大括号 `{…}` 和一些**属性**来创建。一个属性就是一个“键值对”，其中`键`是一个字符串（也可以称为一个“属性名”）， `值`可以是任何东西。

我们可以想象对象就是一个柜子里放着带符号的文件。每一份数据根据它的 key 来存储。这样就很容易通过文件的名字来查找文件或者添加、删除一个文件。

![](object.png)

一个空的对象（“空的柜子”）可以通过两种语法来创建：

```js
let user = new Object(); // "object constructor" syntax
let user = {};  // "object literal" syntax
```

![](object-user-empty.png)

通常，都用大括号`{...}`。那种声明方式被称为**对象字面量**。

## 语法与属性

我们可以立即放一些属性到`{...}`作为“键值对”：

```js
let user = {     // an object
  name: "John",  // by key "name" store value "John"
  age: 30        // by key "age" store value 30
};
```

一个属性在冒号`":"`前有一个键（也被称为“名字”或者“识别符”）和在冒号右边有一个值。

在 `user` 这个对象中，有两个属性：

1. 第一个属性有名字 `"name"` 和它的值 `"John"`。
2. 第二个属性有名字 `"age"` 和它的值 `30`.


最后的 `user` 对象可以想象为一个柜子里装着两个带符号的文件，被标志着 “name” 和 “age”。

![user object](object-user.png)

我们可以在任意时刻添加，删除或者从中读取文件。

属性值可以通过点符号来访问：

```js
// get fields of the object:
alert( user.name ); // John
alert( user.age ); // 30
```

值可以使任意类型。让我们添加一个布尔类型：

```js
user.isAdmin = true;
```

![user object 2](object-user-isadmin.png)

要删除一个属性，我们可以用 `delete` 操作符：

```js
delete user.age;
```

![user object 3](object-user-delete.png)

我们也可以用多个单词来做属性名，但这个时候它们必须用双引号括起来：

```js
let user = {
  name: "John",
  age: 30,
  "likes birds": true  // multiword property name must be quoted
};
```

![](object-user-props.png)


````smart header="尾随逗号"
在属性中的最后一个属性可能以逗号结尾：
```js
let user = {
  name: "John",
  age: 30*!*,*/!*
}
```
这被称为 "trailing" 或者 "hanging" 逗号。它使得添加、删除、移动属性变得更简单，因为所有行都是相似的。
````

## 方括号

对于多单词属性名，通过点来访问就不可行了：

```js run
// this would give a syntax error
user.likes birds = true
```

这是因为点访问需要键是一个有效的变量标识符，就是：不能有空格或者其他的限制的符号。

这里有个替代品“方括号”可以用于所有字符串：


```js run
let user = {};

// set
user["likes birds"] = true;

// get
alert(user["likes birds"]); // true

// delete
delete user["likes birds"];
```

现在所有东西都好了。请注意在方括号里的字符串都被引号括起来（任意括号都可以）。

方括号同时提供用任意表达式来获取属性名的一种方式 —— 相对于用字符串 —— 比如就像下面用变量：

```js
let key = "likes birds";

// same as user["likes birds"] = true;
user[key] = true;
```

这里，变量 `key` 可以在 run-time 时计算或者由用户输入来决定。然后我们用它来访问属性。这样提供了我们巨大的灵活性，而点符号就不能这样类似地用。

比如：

```js run
let user = {
  name: "John",
  age: 30
};

let key = prompt("What do you want to know about the user?", "name");

// access by variable
alert( user[key] ); // John (if enter "name")
```


### 计算属性

我们可以在对象字面量里运用方括号。这个被称为**计算属性**。

比如：

```js run
let fruit = prompt("Which fruit to buy?", "apple");

let bag = {
*!*
  [fruit]: 5, // the name of the property is taken from the variable fruit
*/!*
};

alert( bag.apple ); // 5 if fruit="apple"
```

计算属性的意思很简单：`[fruit]` 意思时这个属性名时从 `fruit` 变量获得.

所以，如果一个访客输入 `"apple"`， `bag` 就会变成 `{apple: 5}`。

本质上，这个效果也是一样的：

```js run
let fruit = prompt("Which fruit to buy?", "apple");
let bag = {};

// take property name from the fruit variable
bag[fruit] = 5;
```

...但是看起来更简洁好看.

我们可以在方括号里用更复杂的表达式：

```js
let fruit = 'apple';
let bag = {
  ['apple' + 'Computers']: 5 // bag.appleComputers = 5
};
```

方括号比点符号更加强大。它允许任意的属性名跟变量。但是它写起来更复杂。

所以大多数时候，当属性名比较简单的时候，我们用点符号。如果我们需要的比较复杂，那么我们就换做用方括号。

````smart header="保留字节可以被用来做属性名"
一个变量不能用程序语言的保留字节来做它的变量名，比如 "for", "let", "return" 之类的。

但是对一个对象属性来说，这里就没有这样的限制，可以使用任意名字：

```js run
let obj = {
  for: 1,
  let: 2,
  return: 3
}

alert( obj.for + obj.let + obj.return );  // 6
```

基本上，所有名字都是允许的，但是这里有个例外：`"__proto__"`，它主要是因为历史原因。比如，我们不能把它赋给一个非对象值：

```js run
let obj = {};
obj.__proto__ = 5;
alert(obj.__proto__); // [object Object], didn't work as intended
```

我们能从代码里看到，将基本类型 `5` 赋值给它是被忽略的。如果我们想存**任意**（用户提供的）键，这样的行为会导致程序错误甚至使程序易受攻击，因为它是不可预测的。还有有种另外一种数据结构 [Map](info:map-set-weakmap-weakset)，我们会在这个章节学习它 <info:map-set-weakmap-weakset>, 它支持任意的键。
````


## 属性值缩写

在实际代码中我们通常用已有的变量来作为属性名对应的值。

比如：

```js run
function makeUser() {
  let name = prompt("Name?");
  let age = prompt("Age?");
  return {
    name: name,
    age: age
  };
}

let user = makeUser("John", 30);
alert(user.name); // John
```

在上述例子中，属性名跟变量一样。这种把变量作为属性的用例非常常见， 所以这里有一种特殊的**属性值缩写**来简写它。

与其写 `name:name` 我们可以只写 `name` 就像这样：

```js
function makeUser(name, age) {
*!*
  return {
    name, // same as name: name
    age   // same as age: age
  };
*/!*
}
```

在同一个对象里我们可以同时用普通的属性和属性缩写：

```js
let user = {
  name,  // same as name:name
  age: 30
};
```

## 存在检查

对象的一个显著特点是它能访问任意属性。即便属性不存在也不会报错！访问一个不存在的属性只是会返回 `undefined`。它给我们提供了一个非常常见的测试一个属性是否存在的方法 —— 去访问它然后用它跟 `undefined` 比较：

```js run
let user = {};

alert( user.noSuchProperty === undefined ); // true means "no such property"
```

还有一个用来检查一个属性是否存在的特殊操作符 `"in"` 。

语法是：

```js
"key" in object
```

比如：

```js run
let user = { name: "John", age: 30 };

alert( "age" in user ); // true, user.age exists
alert( "blabla" in user ); // false, user.blabla doesn't exist
```

请注意 `in` 左边必须是一个**属性名** 。通常是一个带引号的字符串。

如果我们删掉引号，这就意味着被测试的是一个包含真实属性名的变量。比如：

```js run
let user = { age: 30 };

let key = "age";
alert( *!*key*/!* in user ); // true, takes the name from key and checks for such property
```

````smart header="对存 `undefined` 的属性使用 \"in\" "
通常，用严格比较 `"=== undefined"` 来判断是可以的。但是这里有个特殊情况它会失败，然而用 `"in"` 就能正常工作。

就是当一个对象属性存在是，但是它存着 `undefined`：

```js run
let obj = {
  test: undefined
};

alert( obj.test ); // it's undefined, so - no such property?

alert( "test" in obj ); // true, the property does exist!
```


在上述代码中，属性 `obj.test` 理论上是存在的。所以 `in` 操作符能正确工作。

像这种情况一般很少发生，因为 `undefined` 通常不用来赋值。对于“unknown”或者“empty”的值我们通常用 `null`。所以 `in` 操作符是在代码中是比较少见的。
````


## “for..in”循环

要遍历一个对象里的所有键，有一种特殊的循环方式：`for..in`。这是跟我们之前学习的循环结构 `for(;;)` 完全不同的循环方式。

语法：

```js
for(key in object) {
  // executes the body for each key among object properties
}
```

比如，让我们输出 `user` 的所有属性：

```js run
let user = {
  name: "John",
  age: 30,
  isAdmin: true
};

for(let key in user) {
  // keys
  alert( key );  // name, age, isAdmin
  // values for the keys
  alert( user[key] ); // John, 30, true
}
```

注意所有“for”循环结构都允许在循环里声明循环变量，就像在这里的`let key`。

当然，我们也可以使用 `key` 以外的变量。比如 `"for(let prop in obj)"` 也是经常被使用的。


### Ordered like an object

对象是有序的吗？换而言之，如果我们遍历一个对象，我们会以与它们添加时相同的顺序得到属性吗？我们能否依赖它？

简短回答：“以一种特殊形式排序”：整数类型是排序的，而其他的按它们创建时的顺序。细节如下.

举个例子，假设有一个对象存着电话区号：

```js run
let codes = {
  "49": "Germany",
  "41": "Switzerland",
  "44": "Great Britain",
  // ..,
  "1": "USA"
};

*!*
for(let code in codes) {
  alert(code); // 1, 41, 44, 49
}
*/!*
```

这个对象可能被用来提供给用户一串选项。如果我们想做一个主要面向德国用户的网站，那么我们很可能想要 `49` 出现在第一个。

但是如果我们运行这段代码，我们会看到截然不同的结果：

- USA (1) 是第一个
- 然后 Switzerland (41) 等

这些电话区号是按升序排列的，这是因为它们是整数类型。所有我们看到的是 `1, 41, 44, 49`。

````smart header="整数属性？这是啥？"
“整数属性”这个术语意思是一个字符串转换成整数然后再转换回字符串，它仍是相同的。

所以，“49”是一个整数属性名，因为它转换成整数再转换回来，它仍然是相同的。但是“+49”和“1.2”就不是：

```js run
// Math.trunc is a built-in function that removes the decimal part
alert( String(Math.trunc(Number("49"))) ); // "49", same, integer property
alert( String(Math.trunc(Number("+49"))) ); // "49", not same ⇒ not integer property
alert( String(Math.trunc(Number("1.2"))) ); // "1", not same ⇒ not integer property
```
````

...另一方面，如果键不是整数，那么它们就是以创建顺序列出来的，比如：

```js run
let user = {
  name: "John",
  surname: "Smith"
};
user.age = 25; // add one more

*!*
// non-integer properties are listed in the creation order
*/!*
for (let prop in user) {
  alert( prop ); // name, surname, age
}
```

所以，来解决这个电话区号的问题，我们可以把区号变成非整型来“作弊”。在每个区号前面加个 `"+"` 符号就行了。

就像这样：

```js run
let codes = {
  "+49": "Germany",
  "+41": "Switzerland",
  "+44": "Great Britain",
  // ..,
  "+1": "USA"
};

for(let code in codes) {
  alert( +code ); // 49, 41, 44, 1
}
```

现在它就能像预想一样运行了。

## 复制引用

对象与基本类型的根本区别之一是它们通过引用存储和复制。

基本类型：字符串，数字，布尔值 —— 作为整体进行赋值/复制。

例如：

```js
let message = "Hello!";
let phrase = message;
```

因此，我们有两个独立的变量，每个变量都存储字符串 `"Hello!"`.

![](variable-copy-value.png)

对象不是这样的。

**变量存储的不是对象本身，而是“内存中的地址”，换句话说就是“引用”。**

这里是对象的形象例子：

```js
let user = {
  name: "John"
};
```

![](variable-contains-reference.png)

这里，对象被存储在内存的某处。而变量 `user` 有一个“引用”指向它。

**复制对象变量时，复制引用，对象不会被复制。**

如果我们想象一个对象作为一个柜子，那么一个变量就是一个通往它的钥匙。复制对象变量复制的是钥匙，而不是柜子本身。

比如：

```js no-beautify
let user = { name: "John" };

let admin = user; // copy the reference
```

现在我们有两个变量，每个变量都引用同一个对象：

![](variable-copy-reference.png)

现在可以使用任意变量去访问柜子并修改其内容：

```js run
let user = { name: 'John' };

let admin = user;

*!*
admin.name = 'Pete'; // changed by the "admin" reference
*/!*

alert(*!*user.name*/!*); // 'Pete', changes are seen from the "user" reference
```

上面的例子证明只有一个对象。就像我们有个柜子，柜子有两个钥匙一样，使用其中一个（`admin`）进入它。然后，如果我们稍后使用另一个键（`user`），我们看到更改。

### 比较引用

等于符号`==`和严格等于符号`===`对对象来说是完全一样。

**两个对象只有在它们是同一个对象时才相等。**

比如，两个变量引用同一个对象，它们是相等的：

```js run
let a = {};
let b = a; // copy the reference

alert( a == b ); // true, both variables reference the same object
alert( a === b ); // true
```

而这两个独立的对象并不相等，即使两者都是空的：

```js run
let a = {};
let b = {}; // two independent objects

alert( a == b ); // false
```

对于像 `obj1> obj2` 这样的比较，或者与基本类型进行比较 `obj == 5`，对象会被转换成基本类型。我们将会很快研究对象转换是如何工作的，但是说实话，这样的比较很少需要，通常是编程错误的结果。

### Const 对象

声明 `const` 的对象**可以**被改变.

比如：

```js run
const user = {
  name: "John"
};

*!*
user.age = 25; // (*)
*/!*

alert(user.age); // 25
```

看起来 `(*)` 这行可能会报错，但实际上没有，这里完全没有问题。这是因为 `const` 只是固定 `user` 的值本身。而 `user` 一直存储对同一个对象的引用。`(*)` 这行**进入**对象，没有重新对 `user` 赋值。

如果我们尝试将 `user` 设置为别的东西，`const` 会给出一个错误，例如：

```js run
const user = {
  name: "John"
};

*!*
// Error (can't reassign user)
*/!*
user = {
  name: "Pete"
};
```

...但是如果我们想要创建常量对象属性怎么办？就是当使用 `user.age = 25` 会报错。这也是可以的。我们会在章节 <info:property-descriptors> 中介绍。

## 克隆和合并，Object.assign

因此，复制对象变量会创建一个对同一个对象的引用。

但是如果我们需要复制一个对象呢？创建一个独立的副本，一个克隆？

这也是可行的，但有点困难，因为在JavaScript中没有内置的方法。实际上，这很少需要，大多数时候通过参考复制的。

但是，如果我们真的想要这样做，那么我们需要创建一个新对象，并通过遍历其属性并在基础类型的级别上复制它们来复制现有对象的结构。

像这样：

```js run
let user = {
  name: "John",
  age: 30
};

*!*
let clone = {}; // the new empty object

// let's copy all user properties into it
for (let key in user) {
  clone[key] = user[key];
}
*/!*

// now clone is a fully independant clone
clone.name = "Pete"; // changed the data in it

alert( user.name ); // still John in the original object
```

我们也可以使用 [Object.assign](mdn:js/Object/assign) 的方法。

语法是：

```js
Object.assign(dest[, src1, src2, src3...])
```

- 参数 `dest`，和 `src1, ..., srcN` （可以有任意多个）是对象。
- 它将所有对象 `src1, ..., srcN` 的属性复制到 `dest`中。换句话说，从第二个开始的所有参数的属性被复制到第一个。然后它返回 `dest`。

例如，我们可以使用它来将多个对象合并成一个：

```js
let user = { name: "John" };

let permissions1 = { canView: true };
let permissions2 = { canEdit: true };

*!*
// copies all properties from permissions1 and permissions2 into user
Object.assign(user, permissions1, permissions2);
*/!*

// now user = { name: "John", canView: true, canEdit: true }
```

如果接收对象（`user`）已经具有相同的命名属性，它将被覆盖：

```js
let user = { name: "John" };

// overwrite name, add isAdmin
Object.assign(user, { name: "Pete", isAdmin: true });

// now user = { name: "Pete", isAdmin: true }
```

我们也可以使用 `Object.assign` 代替循环进行简单的克隆：

```js
let user = {
  name: "John",
  age: 30
};

*!*
let clone = Object.assign({}, user);
*/!*
```

它将`user`的所有属性复制到空对象中并返回。其实，和循环一样，但更短更简洁。

直到现在我们都是假设 `user` 的所有属性都是基础类型的。但属性可以引用其他对象。他们怎么办？

就像这样：

```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

alert( user.sizes.height ); // 182
```

现在复制`clone.sizes = user.sizes`是不够的，因为`user.sizes`是一个对象，它将被复制引用。 所以 `clone` 和 `user` 会共享同一个 `sizes`：

像这样：

```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = Object.assign({}, user);

alert( user.sizes === clone.sizes ); // true, same object

// user and clone share sizes
user.sizes.width++;       // change a property from one place
alert(clone.sizes.width); // 51, see the result from the other one
```

为了解决这个问题，我们应该使用之前的克隆循环来检查每个 `user[key]` 的值，如果它是个对象，那么也复制它的结构。这被称为“深克隆”。

有一种用于深克隆的标准算法，它能处理上述情况和更复杂的情况，称为 [Structured cloning algorithm](https://w3c.github.io/html/infrastructure.html#internal-structured-cloning-algorithm)。 为了不重新造轮子，我们可以使用JavaScript库中现有的实现 [lodash](https://lodash.com)，方法名叫做 [_.cloneDeep(obj)](https://lodash.com/docs#cloneDeep).



## 总结

对象是具有几个特殊功能的关联数组。

他们存储属性（键值对），其中：
- 属性键必须是字符串或符号（通常是字符串）。
- 值可以是任何类型。

访问一个属性，我们可以用：
- 点符号： `obj.property`。
- 方括号 `obj["property"]`。方括号 方括号允许从变量中取出键，如 `obj[varWithKey]`。

其他的操作符：
- 删除一个属性：`delete obj.prop`。
- 检查是否存在具有给定键的属性: `"key" in obj`。
- 遍历一个对象：`for(let key in obj)` 循环。

对象是通过引用来赋值和拷贝的。换句话说，一个变量不存储“对象值”，而是存储该值的“引用”（内存中的地址）。所以复制这样一个变量或者把它作为一个函数参数来传递它来复制的是引用而不是对象。所有通过复制引用的操作（如添加/删除属性）都在相同的单个对象上执行。

做一个“真正的副本”（一个克隆），我们可以使用 `Object.assign` 或者  [_.cloneDeep(obj)](https://lodash.com/docs#cloneDeep).

我们在本章中研究的内容被称为“纯对象”，或者是 `Object`.

JavaScript中还有许多其他类型的对象：

- `Array` 用来存储有序数据集合，
- `Date` 用来存储有关日期和时间的信息，
- `Error` 用来存储关于错误的信息。
- ...等等。


他们有他们的特点，我们稍后研究。有时人们会说“数组类型”或“日期类型”，但它们不是自己的类型，而是属于单个的“对象”数据类型。他们以各种方式延伸。

JavaScript中的对象非常强大。在这里，我们只是涉猎了话题的表面。在教程的进一步部分，我们将密切地运用对象并更多地了解它们。
