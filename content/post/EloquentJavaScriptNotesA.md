---
title: Eloquent JavaScript笔记：迭代器
date: 2020-01-05 16:23:34
tags:
---

## 迭代器是做什么的

迭代（Iteration）是我们在写程序的时候很常见的操作，在没有迭代器之前，如果我们想要迭代一个可以迭代的对象，可以使用循环来实现：

```javascript
for (let index = 0; index < array.length; index++) {
  //do something
}
```

在 ECMAScript 2015 标准中引入了一种语法叫做`for ... of`循环，用来迭代一个可以被迭代的对象，比如数组，字符串等，那么如何使我们自己创建的类，对象等可以迭代呢？一个可行的办法是创建相应的迭代器。

## 实现迭代器

```javascript
class Group {
  constructor() {
    this.group = [];
  }

  add(item) {
    if (this.group.indexOf(item) === -1) {
      this.group.push(item);
    }
  }
  delete(item) {
    //....
  }
  has(item) {
    //...
  }

  static from(iteraObj) {
    let res = new Group();
    for (let item of iteraObj) {
      res.add(item);
    }
    return res;
  }

  [Symbol.iterator]() {
    return new GroupIterator(this);
  }
}

class GroupIterator {
  //由上文可知，此构造函数的参数是this,也就是说是一个类
  constructor(group) {
    this.index = 0;
    this.group = group;
  }
  next() {
    //要访问的是这个类生成的group，(this.group).group所指向的是Group的构造函数生成的那个group
    if (this.index >= this.group.group.length) {
      return { done: true };
    }
    let value = this.group.group[this.index];
    this.index++;
    return { value: value, done: false };
  }
}
console.log(Group.from(["a", "b", "c"]));
//Group { group: [ 'a', 'b', 'c' ] }
for (let value of Group.from(["a", "b", "c"])) {
  console.log(value);
}
// → a
// → b
// → c
```

上面的代码创建了一个类`Group`，这个类返回一个对象，对象的键值`group`是一个数组，由创建这个类时的`from`静态方法生成，直接迭代这个对象是不可以的，因为它只有一个键值，所以可以通过创建一个迭代器`GroupIterator`来使这个对象可以迭代，迭代的内容是`group`键值对应的数组里的内容。  
首先我们在原类内创建了一个指向迭代器的符号：

```javascript
[Symbol.iterator]() {
    return new GroupIterator(this);
  }
```

这个符号告诉`for...of`方法，如果需要迭代这个类，则将这个类传入对应的迭代器。  

``` javascript
class GroupIterator {
  //由上文可知，此构造函数的参数是this,也就是说是一个类
  constructor(group) {
    this.group = group;
    this.index = 0;
  }
  next() {
    //要访问的是这个类生成的group，(this.group).group所指向的是Group的构造函数生成的那个group
    if (this.index >= this.group.group.length) {
      return { done: true };
    }
    let value = this.group.group[this.index];
    this.index++;
    return { value: value, done: false };
  }
}
```

迭代器拥有一个构造函数，当他接收一个`Group`类的时候，首先将类传入迭代器，然后初始化当前索引为0，迭代器需要一个`next()`方法，这个方法返回一个对象，对象包含迭代器当前返回的值和是否迭代完成的`done`属性，当`done`为true时则停止迭代，在`next()`内可以实现自定义迭代器返回的值。上面代码则是返回`group`键值对应的数组内的每一项。