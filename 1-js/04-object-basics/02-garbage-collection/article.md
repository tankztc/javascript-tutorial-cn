# 垃圾回收机制

JavaScript中的内存管理会自动地、悄悄地执行的。我们创建基础类型，对象，函数...所有这些都要占用内存的。

当有些东西不需要了会发生什么？JavaScript引擎是怎么发现它们并清理它们的呢？

[cut]

## 可达性

JavaScript中的内存管理的主要概念是**可达性**。

简单地说，“可达到的”的值是那些可以访问到或可以用到的。它们保证被存储在内存中。

1. 这里有一群本质上可达的值，它们是明显不能被删除的。

    比如：

    - 当前函数的局部变量和参数。
    - 当前嵌套调用链上其他函数的变量和参数。
    - 全局变量。
    - （还有一些其他内部的）

    这些值称为**根**。

2. 其他的值如果它能从根到达，被根引用，就被认为可到达的的。

    比如，如果在局部变量中有这么一个对象，它有个属性引用其他对象，那么它就被认为是可达的。同时那些它引用的对象 —— 也是可达的。接下来是详细的例子。

JavaScript引擎中有一个后台进程被称为 [garbage collector](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)). 它监视所有对象并删除那些不可达到的对象。

## 一个简单的例子

这是最简单的例子：

```js
// user has a reference to the object
let user = {
  name: "John"
};
```

![](memory-user-john.png)

这里的箭头描绘了一个对象引用。 全局变量 `"user"` 引用对象 `{name: "John"}` （我们简称它为 John）。 John的属性 `"name"`存着一个基础类型, 所以它被画在对象里的。

如果 `user` 的值被覆盖了, 那么这个引用就丢失了：

```js
user = null;
```

![](memory-user-john-lost.png)

现在 John 变得无法到达。没有办法访问它，没有引用它。Garbage collector 将数据当垃圾并释放内存。

## 两个引用

现在我们假设我们把`user`的引用复制到`admin`：

```js
// user has a reference to the object
let user = {
  name: "John"
};

*!*
let admin = user;
*/!*
```

![](memory-user-john-admin.png)

现在如果我们这样做：

```js
user = null;
```


...然后对象仍然可以通过 `admin` 全局变量访问，所以它在内存中。如果我们也覆盖 `admin`，那么它可以被删除。

## 互连对象

现在是一个比较复杂的例子。这是个家庭：

```js
function marry(man, woman) {
  woman.husband = man;
  man.wife = woman;

  return {
    father: man,
    mother: woman
  }
}

let family = marry({
  name: "John"
}, {
  name: "Ann"
});
```


函数`marry`通过给它们互相引用来“结合”两个对象，并返回一个包含它们的新对象。

最后的内存结构：

![](family.png)

到目前为止，所有的对象都是可以访问的。

现在我们来删除两个引用：

```js
delete family.father;
delete family.mother.husband;
```

![](family-delete-refs.png)

仅删除这两个引用中的一个是不够的，因为所有对象仍然可以访问。

但是如果我们删除这两个，那么我们可以看到 John 已经没有传入的引用：

![](family-no-father.png)

传出的参考无关紧要。只有传入的可以使对象可达。所以，John 现在是无法访问的，并且将被从内存中删除，所有它的数据也变得无法访问。

garbage collection 后：

![](family-no-father-2.png)

## 不可达到的岛屿

有可能整个互连对象的岛屿变得不可达，然后从内存中移除。

源对象与上述相同。然后：

```js
family = null;
```

内存变成：

![](family-no-family.png)

此示例演示了可达性概念的重要性。

很明显，John 和 Ann 仍然是链起来的，都有传入的引用。但这还不够。

前面的 `"family"` 对象已经从根脱离出来，没有任何的引用，所以整个岛屿都无法访问，将被删除。

## 内部算法

基本的垃圾回收算法称为“标记扫描”。

定期执行以下“垃圾回收”步骤：

- 垃圾回收器找到根，并标记（记住）它们。
- 然后它访问并标记所有根的引用。
- 然后它又访问这些被标记的对象同时标记**它们**的引用。所有被访问过的对象会被记住，不会访问同一个对象两次。
- ...一直遍历直到所有（从根能到达的）的引用都被访问过。
- 所有除了被标记的对象都被清除。

例如，如果我们的对象结构长得想这样：

![](garbage-collection-1.png)

我们可以清楚地看到在右边有个“不可到达的岛屿”。现在让我们看看“标记扫描”垃圾回收器是如何处理它的。

第一步标记根：

![](garbage-collection-2.png)

然后它们的引用被标记：

![](garbage-collection-3.png)

...And their refences, while possible:

![](garbage-collection-4.png)

现在那些在这个过程中没有被访问的对象被视为不可到达的同时会被清除：

![](garbage-collection-5.png)

这就是垃圾回收的工作原理。

JavaScript 引擎应用许多优化来使它运行得更快并不影响执行。

一些优化：

- **分代收集** —— 对象被分为两组：“新的”跟“旧的”。许多对象出现，完成它们的工作，快速死亡，所以它们可以积极地进行清理。那些生存足够长的，就变“老”了。
- **渐进式收集** —— 如果有很多对象，而我们尝试一次性走并标记整个对象集，这可能需要花一些时间，并会在执行中引入可见的延迟。因此，引擎试图将工作分成几部分。然后一次执行一个。这需要在它们中花费些额外的簿记来跟踪变化。
- **空闲时间收集** —— 垃圾回收器尝试仅在 CPU 空闲时运行，以减少对程序执行可能造成的影响。

垃圾回收算法还有其他的方式和方法。虽然我想尽可能的多描叙它们，但是我必须暂停，因为不同的引擎运用不同的技巧和方法。

而且 —— 更重要的是，随着引擎的发展，这些东西也会发生变化，所以在没有真正需要的情况下，“提前”深入探究可能是不值得的。当然，除非是因为纯粹的兴趣，那么下面会有一些链接。

## 总结

需要知道的些主要事情：

- 垃圾回收是自动执行的。我们不能强制执行它或终止它。
- 当对象可以到达时，会被保留在内存里。
- 被引用了和（从根目录）可到达是不同：一组互连对象也可以变得无法到达。

现代引擎都运用高级的垃圾回收算法。

一本通用书 "The Garbage Collection Handbook: The Art of Automatic Memory Management" (R. Jones at al) 覆盖了其中一些算法。

如果您熟悉低级编程，则有关 V8 垃圾回收器的更详细信息在本文中 [A tour of V8: Garbage Collection](http://jayconrod.com/posts/55/a-tour-of-v8-garbage-collection). 

[V8 blog](http://v8project.blogspot.com/) 还会不时发布有关内存管理的变更的文章。 当然，要学习垃圾回收，你最好通过学习 V8 的内部实现并阅读 [Vyacheslav Egorov](http://mrale.ph) 的博客，他是其中一位 V8 工程师。 who worked as one of V8 engineers. 我拿 “V8" 来做例子讲，是因为它是被互联网文章提及最全的。对于其他的引擎，许多实现方式都是类似的，但垃圾回收却各有各的不同。

当你需要底层优化的时候，深入了解引擎的知识是有很大帮助的。在熟悉了语言之后，把这个作为下一步的目标会是明智的。
