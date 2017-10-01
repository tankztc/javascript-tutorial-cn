# 对象方法，“this”

通常创建对象来表示现实世界的实体，如用户，订单等：

```js
let user = {
  name: "John",
  age: 30
};
```

而且，在现实世界中，用户可以**行动**：从购物车中选择一些东西，登录，注销等。

Actions are represented in JavaScript by functions in properties.

[cut]

## 方法举例

开始，让我们教`user`说你好：

```js run
let user = {
  name: "John",
  age: 30
};

*!*
user.sayHi = function() {
  alert("Hello!");
};
*/!*

user.sayHi(); // Hello!
```

在这里，我们刚刚使用一个函数表达式来创建函数并将其分配给对象的属性`user.sayHi`。

然后我们就可以调用它。这个 user 现在可以说话了！

当一个函数作为一个对象的属性时，它被称为**方法**。

所以，这里我们得到了对象`user`的一个方法`sayHi`。

当然，我们可以使用一个预先声明的函数作为一个方法，像这样：

```js run
let user = {
  // ...
};

*!*
// first, declare
function sayHi() {
  alert("Hello!");
};

// then add as a method
user.sayHi = sayHi;
*/!*

user.sayHi(); // Hello!
```

```smart header="Object-oriented programming"
When we write our code using objects to represent entities, that's called an [object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming), in short: "OOP".

OOP is a big thing, an interesting science of its own. How to choose the right entities? How to organize the interaction between them? That's architecture, and there are great books on that topic, like "Design Patterns: Elements of Reusable Object-Oriented Software" by E.Gamma, R.Helm, R.Johnson, J.Vissides or "Object-Oriented Analysis and Design with Applications" by G.Booch, and more. We'll scratch the surface of that topic later in the chapter <info:object-oriented-programming>.
```
### 方法简写

对象文字中的方法存在较短的语法：

```js
// these objects do the same

let user = {
  sayHi: function() {
    alert("Hello");
  }
};

// method shorthand looks better, right?
let user = {
*!*
  sayHi() { // same as "sayHi: function()"
*/!*
    alert("Hello");
  }
};
```

如上所示，我们可以省略`"function"`而只写`sayHi()`。

说实话，符号并不完全相同。与对象继承有微妙差异（稍后将介绍），但是现在它们并不重要。在几乎所有情况下，较短的语法是首选的。

## 方法中"this"

通常，对象方法需要访问存储在对象中的信息来完成其工作。

例如，`user.sayHi()`中的代码可能需要`user`的名字。

**要访问对象，方法可以使用`this`关键字。**

`this`的值是哪个“在点符号前”的对象，那个调用方法的对象。

比如：

```js run
let user = {
  name: "John",
  age: 30,

  sayHi() {
*!*
    alert(this.name);
*/!*
  }

};

user.sayHi(); // John
```

这里在执行`user.sayHi()`过程中，`this`的值将会是`user`。

从技术上讲，也可以不用`this`来访问对象，通过外部变量引用它：

```js
let user = {
  name: "John",
  age: 30,

  sayHi() {
*!*
    alert(user.name); // "user" instead of "this"
*/!*
  }

};
```

...但这样的代码是不可靠的。如果我们决定将`user`复制到另一个变量，例如`admin = user`并用别的东西覆盖`user`，那么它会访问错误的对象。

如下所示：

```js run
let user = {
  name: "John",
  age: 30,

  sayHi() {
*!*
    alert( user.name ); // leads to an error
*/!*
  }

};


let admin = user;
user = null; // overwrite to make things obvious

admin.sayHi(); // Whoops! inside sayHi(), the old name is used! error!
```

如果我们在`alert`中使用`this.name`代替`user.name`，那么代码就没有问题了。

## "this" 不绑定

在JavaScript中，“this”关键字与大多数其他编程语言不同。首先，它可以用于任何函数。

像这样的代码中没有语法错误：

```js
function sayHi() {
  alert( *!*this*/!*.name );
}
```

`this`的值在 run-time 时计算，它可以是任何东西。

例如，当从不同对象调用时，相同函数可能具有不同的“this”：

```js run
let user = { name: "John" };
let admin = { name: "Admin" };

function sayHi() {
  alert( this.name );
}

*!*
// use the same functions in two objects
user.f = sayHi;
admin.f = sayHi;
*/!*

// these calls have different this
// "this" inside the function is the object "before the dot"
user.f(); // John  (this == user)
admin.f(); // Admin  (this == admin)

admin['f'](); // Admin (dot or square brackets access the method – doesn't matter)
```

实际上，我们可以在没有对象的情况下调用该函数：

```js run
function sayHi() {
  alert(this);
}

sayHi(); // undefined
```

在严格模式这种情况下`this`是`undefined`。如果我们尝试访问`this.name`，就会有错误发生。

在非严格模式下（如果他忘记`use strict`），`this`的值在那种情况下会是**全局对象**（在浏览器中的`window`，我们会在后面讲到它）。这是个`"use strict"`修复的历史行为。

请注意，通常使用`this`而不使用对象的函数调用是不正常，是编程错误。如果一个函数具有`this`，那么通常意味着在一个对象的上下文中被调用。

```smart header="The consequences of unbound `this`"
If you come from another programming language, then you are probably used to the idea of a "bound `this`", where methods defined in an object always have `this` referencing that object.

In JavaScript `this` is "free", its value is evaluated at call-time and does not depend on where the method was declared, but rather on what's the object "before the dot".

The concept of run-time evaluated `this` has both pluses and minuses. On the one hand, a function can be reused for different objects. On the the other hand, greater flexibility opens a place for mistakes.

Here our position is not to judge whether this language design decision is good or bad. We'll understand how to work with it, how to get benefits and evade problems.
```

## 内部：参考类型

```warn header="In-depth language feature"
This section covers an advanced topic, to understand certain edge-cases better.

If you want to go on faster, it can be skipped or postponed.
```

一个复杂的方法调用可能会丢失`this`，例如：

```js run
let user = {
  name: "John",
  hi() { alert(this.name); },
  bye() { alert("Bye"); }
};

user.hi(); // John (the simple call works)

*!*
// now let's call user.hi or user.bye depending on the name
(user.name == "John" ? user.hi : user.bye)(); // Error!
*/!*
```

在最后一行有一个三元运算符选择`user.hi`或`user.bye`。在这种情况下，结果是`user.hi`。

该方法立即用括号`()`调用。但它没有正常运作！

你能看到调用结果有错误，因为调用中的`"this"`的值变成了`undefined`。

这个有用（对象点方法）：This works (object dot method):

```js
user.hi();
```

这个则不行（评估方法）：

```js
(user.name == "John" ? user.hi : user.bye)(); // Error!
```

为什么？如果我们想了解为什么会发生这种情况 - 让我们来看看`obj.method()`调用的工作原理。

仔细观察，我们可能会注意到`obj.method()`语句中的两个操作：

1. 首先，点`'.'`取得属性`obj.method`。
2. 然后括号`()`执行它。

所以，`this`相关的信息是怎么从第一部分传到第二部分呢？

如果我们将这些操作放在分开的行上，那么`this`将会丢失：

```js run
let user = {
  name: "John",
  hi() { alert(this.name); }
}

*!*
// split getting and calling the method in two lines
let hi = user.hi;
hi(); // Error, because this is undefined
*/!*
```

这里`hi = user.hi`将函数放入变量中，然后在最后一行完全独立，所以没有`this`。

**为了使`user.hi()`调用工作，JavaScript使用一个技巧 - 点`'.'`返回不是一个函数，而是返回一个特殊的[引用类型](https://tc39.github.io/ecma262/#sec-reference-specification-type)。**

引用类型是“规格类型”。我们不能明确地使用它，但是它被语言内部使用。

引用类型的值是一个三值组合`(base，name，strict)`，其中：

- `base`是那个对象。
- `name`是那个属性。
- `strict`是 true 如果`use strict`生效。

属性访问`user.hi`的结果不是函数，而是引用类型的值。在严格模式`user.hi`是：

```js
// Reference Type value
(user, "hi", true)
```

在引用类型上调用括号`()`时，它们会收到关于对象及其方法的完整信息，并且可以设置正确的`this`（在这种情况下为`= user`）。

任何其他操作，如赋值`hi = user.hi`将引用类型整体丢弃，取`user.hi`的值（一个函数）并传递给它。所以任何进一步的操作“失去”`this`。

所以，结果是，`this`的值只有在函数直接通过点`obj.method()`或者方括号`obj[method]()`（它们做的是同样的事情）调用的时候正确的传入。

## 箭头函数没有 "this"

箭头函数是特别的：他们没有“自己”的`this`。如果我们从箭头函数中引用`this`，它取自外部的“正常”函数。

例如，这里`arrow()`用的`this`是从外部的`user.sayHi()`方法来的：

```js run
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // Ilya
```

这是箭头函数的一个特殊功能，当我们实际上不想要单独的`this`，而是从外部上下文中获取它时，这很有用。稍后在<info:arrow-functions>的章节中，我们将更深入地介绍arrow函数。

## 总结

- 存储在对象属性中的函数称为“方法”。
- 方法允许对象像`object.doSomething()`执行“动作”。
- 方法可以通过`this`引用对象。`this`的值在 run-time 时定义。
- 当一个函数被声明时，它可以使用`this`，但是`this`没有任何值，直到函数被调用。
- 函数可以在对象间拷贝。
- 当一个函数在“方法”语法中被调用：`object.method()`时，`this`在调用期间的值是`object`。

请注意，箭头函数是特殊的：它们没有`this`。当`this`在箭头函数内访问时，它将从外部获取。
