
# 符号类型

按照规范，对象属性键可以是字符串类型，也可以是符号类型。不是数字，不是布尔，只有字符串或符号，这两种类型。

到目前为止，我们只看到了字符串类型。现在我们来看看符号类型给我们的优势。

[cut]

## 符号

“符号”值表示具有给定名称的唯一标识符。

可以使用`Symbol(name)`创建此类型的值

```js
// id is a symbol with the name "id"
let id = Symbol("id");
```

符号保证是唯一的。即使我们创建了许多具有相同名称的符号，它们也是不同的值。

例如，这里有两个同名的符号 - 它们不相等：

```js run
let id1 = Symbol("id");
let id2 = Symbol("id");

*!*
alert(id1 == id2); // false
*/!*
```

如果您熟悉 Ruby 或其他具有某种“符号”的语言 - 请不要被误导。 JavaScript 符号是不同的。

````warn header="Symbols don't auto-convert to a string"
Most values in JavaScript support implicit conversion to a string. For instance, we can `alert` almost any value, and it will work. Symbols are special. They don't auto-convert.

For instance, this `alert` will show an error:

```js run
let id = Symbol("id");
*!*
alert(id); // TypeError: Cannot convert a Symbol value to a string
*/!*
```

If we really want to show a symbol, we need to call `.toString()` on it, like here:
```js run
let id = Symbol("id");
*!*
alert(id.toString()); // Symbol(id), now it works
*/!*
```

That's a "language guard" against messing up, because strings and symbols are fundamentally different and should not occasionally convert one into another.
````



## “隐藏”属性

符号允许创建一个对象的“隐藏”属性，其他部分的代码不会不小心访问或覆盖掉。

例如，如果要给对象`user`存储一个“标识符”，我们可以为其创建一个名称为`id`的符号：

```js run
let user = { name: "John" };
let id = Symbol("id");

user[id] = "ID Value";
alert( user[id] ); // we can access the data using the symbol as the key
```

现在让我们想象另一个脚本，为了自己的目的，想在`user`里面有自己的 “id” 属性。这可能是另一个JavaScript库，所以这些脚本对于彼此完全不了解。

没问题。它可以创建自己的`Symbol("id")`。

他们的脚本：

```js
// ...
let id = Symbol("id");

user[id] = "Their id value";
```

这样就不会有冲突，因为符号总是不同的，即使它们有相同的名称。

请注意，如果我们在这种情况下使用字符串`”id“`而不是符号，那么**将**是一个冲突：

```js run
let user = { name: "John" };

// our script uses "id" property
user.id = "ID Value";

// ...if later another script the uses "id" for its purposes...

user.id = "Their id value"
// boom! overwritten! it did not mean to harm the colleague, but did it!
```

### 字面上的符号

如果我们要在对象字面中使用符号，我们需要方括号。

比如这个：

```js
let id = Symbol("id");

let user = {
  name: "John",
*!*
  [id]: 123 // not just "id: 123"
*/!*
};
```
这是因为我们需要变量`id`的值作为键，而不是字符串“id”。

### for..in中符号会被省略

符号属性不参与`for..in`循环。

比如：

```js run
let id = Symbol("id");
let user = {
  name: "John",
  age: 30,
  [id]: 123
};

*!*
for(let key in user) alert(key); // name, age (no symbols)
*/!*

// the direct access by the symbol works
alert( "Direct: " + user[id] );
```

这是通用“隐藏”概念的一部分。如果另一个脚本或库循环了我们的对象，它不会意外地访问符号属性。

相反, [Object.assign](mdn:js/Object/assign) 复制字符串和符号属性：

```js run
let id = Symbol("id");
let user = {
  [id]: 123
};

let clone = Object.assign({}, user);

alert( clone[id] ); // 123
```

这里没有矛盾。这是设计好的。这个想法是当我们克隆一个对象或合并对象时，我们通常需要复制**全部**属性（包括像`id`这样的符号）。

````smart header="Property keys of other types are coerced to strings"
We can only use strings or symbols as keys in objects. Other types are coerced to strings.

For instance:

```js run
let obj = {
  0: "test" // same as "0": "test"
}

// both alerts access the same property (the number 0 is converted to string "0")
alert( obj["0"] ); // test
alert( obj[0] ); // test (same property)
```
````

## 全局符号

通常，所有符号都不同。但有时我们希望相同的符号是一样的。

例如，我们应用程序的不同部分想要访问符号`”id“`，它代表着完全相同的属性。

为了实现这一点，存在一个**全局符号注册表**。我们可以在其中创建符号，并在以后访问它们，并保证同名的重复访问返回完全相同的符号。

在注册表中创建或读取一个符号，可以使用`Symbol.for(name)`。

比如

```js run
// read from the global registry
let name = Symbol.for("name"); // if the symbol did not exist, it is created

// read it again
let nameAgain = Symbol.for("name");

// the same symbol
alert( name === nameAgain ); // true
```

注册表中的符号称为**全局符号**。如果我们想要一个应用程序范围的符号，可以在代码中随处访问 - 这就是它们的用途。

```smart header="That sounds like Ruby"
In some programming languages, like Ruby, there's a single symbol per name.

In JavaScript, as we can see, that's right for global symbols.
```

### Symbol.keyFor

对于全局符号来说，不仅有`Symbol.for(name)`根据 name 返回一个符号，而且还有一个反向调用：`Symbol.keyFor(sym)`，它反过来：根据全局符号返回它的 name。

例如：

```js run
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// get name from symbol
alert( Symbol.keyFor(sym) ); // name
alert( Symbol.keyFor(sym2) ); // id
```

`Symbol.keyFor`内部使用全局符号注册表来查找符号的名称。所以它不适用于非全局符号。如果符号不是全局的，它将无法找到它并返回`undefined`。

例如：

```js run
alert( Symbol.keyFor(Symbol.for("name")) ); // name, global symbol

alert( Symbol.keyFor(Symbol("name2")) ); // undefined, non-global symbol
```

对于非全局符号，它的名称仅用于调试目的。

## System symbols

JavaScript 内部存在许多“系统”符号，我们可以使用它们来微调我们对象的各个方面。

它们罗列在规范的 [Well-known symbols](https://tc39.github.io/ecma262/#sec-well-known-symbols) 表格中：

- `Symbol.hasInstance`
- `Symbol.isConcatSpreadable`
- `Symbol.iterator`
- `Symbol.toPrimitive`
- ...等等。

例如，`Symbol.toPrimitive`可以让将对象转化成原始类型。我们将很快看到它的使用。

其他符号在我们研究相应的语言特征时，也会变得很熟悉。

## 总结

- 符号是唯一标识符的原始类型。
- 符号通过调用`Symbol(name)`创建.
- 如果我们要创建只有那些知道符号才能访问的字段，这时符号就很有用。
- 在`for..in`循环中，符号不会出现。
- 通过`Symbol(name)`创建的符号总是不同的，即使它们具有相同的名称。如果我们想要相同名称的符号，那么我们应该使用全局注册表：`Symbol.for（name）`返回（如果需要，创建）一个带有给定名称的全局符号。多个调用返回相同的符号。
- 有些JavaScript使用的系统符号，可以用`Symbol.*`访问。我们可以用它来改变一些内置的行为。

理论上来说，符号不是100％隐藏。有一个内置的方法[Object.getOwnPropertySymbols(obj)](mdn:js/Object/getOwnPropertySymbols) 可以获取所有符号。还有一个方法叫 [Reflect.ownKeys(obj)](mdn:js/Reflect/ownKeys) 返回对象的**所有**键，包括符号键。所以他们不是真的隐藏。但大多数库，内置方法和语法结构都遵循一个共同的协议。而那些调用上述方法的人应该很了解他在做什么。
