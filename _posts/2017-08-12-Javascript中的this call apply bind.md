---
layout: post
title: 'Javascript中的this,call,apply,bind函数'
date: 2017-08-12 18:45:23 +0800
categories: ['编程']
tags: ['javascript']
author: Alex
noexcerpt: 1
---

# 1 `this`

- 对于顶层对象的概念:
  - 在浏览器中是 window,但是 node 和 Web Worker 里面没有 window。
  - 浏览器和 Web Worker 中 self 也指向顶层对象，但 node 中没有。
  - 在 node 里面，顶层对象是 global，但其它环境不支持。
- **在严格模式下，未指定环境对象而调用函数，则 this 值不会转型为 window/global 等。 除非明确把函数添加到某个对象或者调用 call,apply,bind，否则 this 值将是 undefined**。
- 这里使用的是 node,支持 ES6。运行测试用例`node test.js`。
- 三种调用模式来判断 this 的值。

  ```
  // 方法调用
  // 当一个函数被保存为对象的一个属性时，成为方法。调用时，this指向这个对象

  var handle = {
      name: 'Handle Name',
      init: function() {
          console.log(this.name, this);
      }
  }
  handle.init();  // 输出 Handle Name { name: 'Handle Name', init: [Function: init] }

  // 普通函数调用(非ES6的箭头函数)
  // 在严格模式下，this为undefined，输出 undefined false
  // 在非严格模式下，this指向全局，输出 global, true

  var normal = function () {
      console.log(this, this === global);
  }
  normal();

  // ES6箭头函数
  // this指向调用它的对象

  function Timer() {
      this.s1 = 0,
      this.s2 = 0,

      // 箭头函数，this指向调用其的Timer，这里会改变Timer.s1
      setInterval(() => {
          this.s1++;
          console.log('箭头函数: ', this);    // this: Timer { s1: 11, s2: 0 }
      }, 1000);
      // 指向全局，这里并不会改变Timer.s2
      setInterval(function () {
          this.s2++;
          console.log('普通函数', this);      // this: global
      }, 1000);
  }

  var timer = new Timer();
  setTimeout(() => { console.log('s1: ', timer.s1) }, 3500)   // 输出 s1: 3
  setTimeout(() => { console.log('s2: ', timer.s2) }, 3500)   // 输出 s2: 0
  ```

# 2 `call`与`apply`

- 二者作用差不多，会修改函数体内的`this`指向。

  ```
  function log() {
      console.log(this === global, this === undefined, this);
  }

  var foo = {
      name: 'Foo'
  }

  log();
  log.call(foo);
  log.apply(foo);

  // 严格模式下输出
  // log():           false true undefined
  // log.call(foo):   false false { name: 'Foo' }
  // log.apply(foo):  false false { name: 'Foo' }

  // 非严格模式下输出
  // log():           true false global
  // log.call(foo):   false false { name: 'Foo' }
  // log.apply(foo):  false false { name: 'Foo' }
  ```

  - 以上表明经过 call/apply 处理后，this 均指向了 foo。

- 二者传入参数的方式不同

  - 第一个参数做为当前对象，也就是 this 对象。
    - 不传或者传 null, undefined, 会使得 this 指向 global(非严格模式下)。
  - call: 参数要一个一个全部传入，不许使用 arguments 对象。
  - apply: 参数通过组成数组传入，或使用 arguments 对象。

  ```
  function log(x, y) {
      console.log(this, x, y);
  }

  var a = function(x, y) {
      log.call(a, x, y);          // 输出 1 2
      log.call(a, arguments);     // 输出 { '0': 1, '1': 2 } undefined，不能正确获取参数

      log.apply(a, x, y);         // 报错，需要组成数组
      log.apply(a, arguments);    // 输出 1 2
      log.apply(a, [x, y]);       // 输出 1 2, 通过数组
  }
  a(1, 2);
  ```

# 3 `bind`

- 对于给定的函数，会创建一个绑定函数，这个绑定函数的 this 会指向传入的对象

```
function log(x, y) {
    console.log(this === global, this.name, x, y);
}

var people = {
    name: 'People'
}
// 输出 true undefined, log函数在全局上下文中调用, this指向global
log(1, 2);

// 输出 false 'People'
// log中的this被绑定为people
var foo = log.bind(people);
foo(1, 2);

// 也可以这样
var foo2 = log.bind(people, 1, 2);
foo2();
```

# 4 `一个有意思的示例`

- 来源: [Alex MacCaw 如何面试前端工程师：GitHub 很重要](https://segmentfault.com/a/1190000000375138?page=1)
- 关于 apply

  ```
  function log() {
      var args = Array.prototype.slice.call(arguments);
      args.unshift(new Date().toLocaleString());
      console.log.apply(console, args);
  }

  log('hello', 'world');          // 输出 2017-8-12 18:11:26 hello world
  log('hello');                   // 输出 2017-8-12 18:11:26 hello
  ```

- 关于 bind

  ```
  var User = {
      count: 1,

      getCount: function () {
          return this.count;
      }
  };

  // 函数在User的上下文中执行,this指向User
  console.log(User.getCount()); // 输出 1

  // func在全局上下文中执行,this指向global(非严格模式下)
  var func = User.getCount;
  console.log(func()); // 输出 undefined

  // 绑定了User,this指向User
  var func2 = User.getCount.bind(User);
  console.log(func2()); // 输出 1
  ```

# 5 参考资料

- [JavaScript 语言精粹](https://book.douban.com/subject/3590768/)
- [Alex MacCaw 如何面试前端工程师：GitHub 很重要](https://segmentfault.com/a/1190000000375138?page=1)
