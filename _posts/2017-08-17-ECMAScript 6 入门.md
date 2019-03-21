---
layout: post
title: 'ECMAScript 6 入门'
date: 2017-08-17 11:02:43 +0800
categories: ['读书', '编程']
tags: ['javascript']
author: Alex
permalink: /get-started-es6
---

`ECMAScript 6`（简称`ES6`）是于 2015 年 6 月正式发布的`JavaScript`语言的标准，正式名为`ECMAScript 2015（ES2015`）

# 1 建议

- 建议先看看关于 this, call, apply, bind 相关的内容，后文有涉及
- 参考: [Javascript 中的 this,call,apply,bind 函数](http://alex-my.xyz/js-thi-call-apply-bind)

# 2 let,var,const

- let 用于声明变量，但只在所在的代码块中有效。
- const 定义的变量不可以修改，且必须初始化, 且只在所在的代码块中有效。
- const 实际上是变量所指向的那个内存地址不可修改，内存地址如果是一个指针，那么指针所指向的数据是可以修改的。
- var 声明的变量没有初始化不会报错，会输出 undefined。
- var 没有声明变量就是用也不会报错，因为存在变量提升。
- 由于 var 全局范围有效，`i`最终值为 10，所以无论是 a[1]()还是 a[2]()，输出的统统都是 10。

  ```javascript
  var a = [];
  for (var i = 0; i < 10; i++) {
    a[i] = function() {
      console.log(i);
    };
  }
  a[6](); // 10
  ```

- let 只在本轮循环中有效，所以每一次循环的`i`变量其实都是一个新的变量，所以 a[1]()=1,a[2]()=2。

  ```javascript
  var a = [];
  for (let i = 0; i < 10; i++) {
    a[i] = function() {
      console.log(i);
    };
  }
  a[6](); // 6
  ```

- `for`循环中的循环部分是一个父作用域，循环体是一个子作用域。
- 二者如果声明了相同名称的变量，则这两个变量是独立的。
- 如果在`let i = 'abc'`前添加`console.log(i)`则会报错。
- 因为只要快级作用域内存在`let`命令，它所声明的变量的绑定了这个区域。在这个区域类，这个变量就是被`let`修饰的。在声明之前就是用就会报错。同理于 const。
  ```javascript
  for (let i = 0; i < 3; ++i) {
    let i = 'abc';
    console.log(i);
  }
  // 连续输出三个abc
  // abc
  // abc
  // abc
  ```
- let 不允许在同一个作用域内，重复声明同一个变量。同理于 const。

  ```javascript
  let a = 1;
  var a = 2; // 报错, a已经定义了

  console.log(a);
  ```

# 3 解构赋值

- 变量解构成功的示例

  ```javascript
  let [a, b, c] = [1, 2, 3];
  let [a, , c] = [1, 2, 3]; // a=1,c=3
  let [a, b, c] = [1]; // a=1,b=undefined,c=undefined
  ```

- 变量解构失败的示例

  ```javascript
  let [a] = 1;
  let [a] = null;
  let [a] = {};
  ```

  - 但凡右边不是可以遍历的解构(带有 Iterator 接口)，均不能解构。

- 变量解构允许有默认值

  ```javascript
  let [a = 1] = []; // a = 1
  let [a = 1] = [null]; // a = null
  let [a = 1] = [undefined]; // a = 1

  function f() {
    console.log('function f');
  }
  let [a = f()] = [1]; // 并不会执行f()
  ```

  - 必须 === undefined, 才会使用到默认值。
  - 默认值是惰性求值的，只有使用到了才会去调用。

- 对象解构赋值

  ```javascript
  let { A, B } = { A: 'a', B: 'b' }; // A = a, B = b
  let { A: a, B: b } = { A: 'Aa', B: 'Bb' }; // a = Aa, b = Bb
  ```

  - 在 A: a 中，A 属于模式，a 才是要被赋值的变量。

- 对象解构允许有默认值

  ```javascript
  let { A: a = 'aa', B: b = 'bb' } = {}; // a = aa, b = bb
  ```

- 字符串解构

  ```javascript
  let [a, b, c, d, e, f] = 'hello world'; // a = h, b = e, c = l,...
  let { length: len } = 'hello world'; // len = 11, 字符串会被转为一个对象，对象中有length属性。
  ```

- 只要右边不是数组或者对象，都会尝试转为对象，然后再解构。
- 用途

  - 从函数中返回多个值，就如 python 一样
  - 提取 JSON 数据

    ```javascript
    let jsonData = {
      id: 1,
      age: 20,
    };
    let { id, age } = jsonData;
    ```

  - 遍历 Map

    ```javascript
    var map = new Map();

    map.set('first', 'hello');
    map.set('second', 'world');

    for (let [key, value] of map) {
      console.log(`key ${key} => value ${value}`);
    }
    for (let [key] of map) {
      console.log(`key ${key}`);
    }
    for (let [, value] of map) {
      console.log(`value ${value}`);
    }
    ```

# 4 字符串

- 遍历字符串
  ```javascript
  for (let code of 'hello world') {
    console.log(code);
  }
  ```
- at

  ```javascript
  console.log('abc'.charAt(1)); // b
  console.log('abc'.charCodeAt(1)); // 98
  ```

- includes(), startsWith(), endsWith()

  ```javascript
  let fileName = 'test.js';
  console.log(fileName.includes('es')); // true, 是否找到了参数es
  console.log(fileName.startsWith('t')); // true，是否以参数t开头
  console.log(fileName.endsWith('.js')); // true, 是否以参数.js结尾
  ```

  - 以上三个函数都支持第二个参数，表示开始搜索的位置。

- repeat()

  ```javascript
  console.log('a'.repeat(3)); // aaa, 返回一个新字符串，表示将原字符串重复n次
  console.log('a'.repeat(2.9)); // aa, 小数点会被去除
  // console.log('a'.repeat(-2.4));   // 报错, 取证后是负数
  console.log('a'.repeat('3')); // aaa, 字符串将会被转为数字
  console.log('a'.repeat(NaN)); // NaN等同于0
  ```

- 模版字符串

  - 现在可以将大段字符串这样写

    ```javascript
    let a = `
        <h1>Index</h1>
        <p>Hello</p>
    `.trim(); // 输出的字符串前后都有一个换行,可以用trim去除
    ```

  - 通过\${}嵌入变量

    ```javascript
    let name = 'Alex';
    console.log(`Hello ${name}`);
    ```

  - 通过\${}嵌入函数

    ```javascript
    function func() {
      return 'hello world';
    }

    console.log(`say: ${func()}`);
    ```

  - 通过\${}嵌入对象
    - 将会调用对象的 toString 方法。
  - 如果\${}内部为字符串，将会原样输出。

    ```javascript
    console.log(`say: ${'hello world'}`); // say: hello world
    ```

# 5 函数

- 函数参数的默认值

  - ES6 之前，不能直接为函数参数指定默认值，但可以用以下方法

    ```javascript
    function log(x, y) {
      x = x || 'Hello'; // x, y 为false, 0之类的就会判断错误
      y = y || 'World';
      console.log(x, y);
    }

    // 修改成这样就不会误判false, 0之类的了
    function log(x, y) {
      if (x === undefined) {
        x = 'Hello';
      }
      if (y === undefined) {
        y = 'World';
      }
      console.log(x, y);
    }
    ```

  - ES6 可以使用默认函数

    ```javascript
    function log(x = 1, y = 2) {
      console.log(x, y);
    }

    log(); // 输出 1 2
    log('a'); // 输出 'a' 2
    ```

    - 如果想让 x 使用默认值, y 使用输入值，需要写成 log(undefined, 'b'), 如果与解构赋值配合，会更好。

  - 解构赋值

    ```javascript
    function log({ x = 1, y = 2 }) {
      console.log(x, y);
    }
    log({}); // 输出 1 2
    log(); // 报错，不能省略 {}
    log({ y: 'b' }); // 输出 1 'b'
    ```

    解构赋值配合参数默认值

    ```javascript
    function log({ x = 1, y = 2 } = {}) {
      console.log(x, y);
    }

    log(); // 不会报错，输出 1 2
    ```

- length

  - length 返回该函数预期传入的参数个数，如果某个参数指定了默认值，这个参数以及之后的参数都不计入了。
  - 不包括 rest 参数

  ```javascript
  console.log(function log(x, y = 2) {}.length); // 输出1
  console.log(function log(x, y = 2, z) {}.length); // 输出1
  console.log(function log(x, y, ...params) {}.length); // 输出2,不包括rest参数
  ```

- 不可省略参数

  ```javascript
  function throwIfMissing() {
    throw new Error('Missing parameter.');
  }

  function log(x = throwIfMissing()) {
    console.log(x);
  }

  log(); // 报错，抛出错误 Missing paramter
  ```

- rest 参数

  ```javascript
  function log(...params) {
    console.log(params);
    for (let p of params) {
      console.log(p);
    }
  }

  log(1, 'hello', 'world');

  // 输出
  // [ 1, 'hello', 'world' ]
  // 1
  // hello
  // world
  ```

  - rest 参数之后不能再跟其它参数了，必须是最后一个参数，会报错

    ```javascript
    function log(...params, x = 1) {
    }
    // 报错 Rest parameter must be last formal parameter
    ```

  - length 不包括 rest 参数。

- name 属性

  - 在 ES6 中, 函数的 name 属性会返回该函数的函数名。

    ```javascript
    // 返回函数名称
    console.log(function log() {}.name);
    // 匿名函数
    var f = function() {};
    console.log(f.name); // 输出 f
    // 具名函数
    var f = function log() {};
    console.log(f.name); // 输出具名函数的名称 log
    // Function
    console.log(new Function().name); // 输出 anonymous(匿名的)
    // bind
    console.log(f.bind().name); // 输出 anonymous
    ```

- 箭头函数

  - ES6 允许使用`=>`来定义函数。

    ```javascript
    // 相当于 function f(v) { return v + 5; };
    var f = v => v + 5;

    // 不带参数或者带多个参数可以使用()
    var f = () => 'hello world';
    var f = (x, y) => x + y;

    // 简化回调函数
    [1, 2, 3].map(function(x) {
      return x * x;
    });
    [1, 2, 3].map(x => x * x);

    // 与rest参数结合
    const headAndTail = (head, ...tail) => [head, tail];
    console.log(headAndTail(1, 2, 3, 4, 5)); // 输出 [ 1, [ 2, 3, 4, 5 ] ]
    ```

  - 箭头函数不可以做为构造函数，也就是不可以使用`new`命令。
  - 箭头函数内不可以使用`yield`函数，因此不可以做为`Generator`函数。
  - 在箭头函数中 this 对象是固定的

    ```javascript
    function foo() {
      setTimeout(() => {
        console.log('id: ', this.id);
      }, 1000);
    }
    var id = 21;
    // call: 相当于把foo中的函数给{id:42}这个对象来执行
    foo.call({ id: 42 });
    ```

    ```javascript
    function Timer() {
      this.s1 = 0;
      this.s2 = 0;

      // 箭头函数中的this绑定定义时所在的作用域，Timer函数
      setInterval(() => this.s1++, 1000);
      // 普通函数中的this绑定运行时所在的作用域，全局对象
      // 因此没有改变Timer.s2的值
      setInterval(function() {
        this.s2++;
      }, 1000);
    }

    var timer = new Timer();

    setTimeout(() => console.log('s1: ', timer.s1), 3500); // s1: 3
    setTimeout(() => console.log('s2: ', timer.s2), 3500); // s2: 0
    ```

# 6 数组

- 扩展运算符 ...

  - 相当于 rest 参数的逆运算，将一个数组转为用逗号分割的参数序列

    ```javascript
    console.log(1, ...[2, 3, 4, 5], 6); // 输出 1 2 3 4 5 6
    ```

  - 提供了数组合并的新方法

    ```javascript
    var a = [1, 2];
    var b = [3, 4];
    var c = [5, 6];

    // ES5
    console.log(a.concat(b).concat(c));
    // ES6
    console.log([...a, ...b, ...c]);
    ```

  - 与解构赋值结合

    ```javascript
    var [first, ...rest] = [1, 2, 3, 4, 5];
    console.log(first, rest);   // 1 [ 2, 3, 4, 5 ]

    var [first, ...rest] = [1];
    console.log(first, rest);       // 1 []

    var [first, ...rest] = [];
    console.log(first, rest);       // undefined []

    var [...rest, first] = [];
    console.log(first, rest);       // 报错: Rest element must be last element
    ```

- Array.from()

  - 该方法可以将两类对象转为数组：类似数组的对象(arrayLike)和可遍历的对象(iterable)(包括 ES6 新增的 Set 和 Map)
  - arrayLike 对象

    ```javascript
    interface ArrayLike<T> {
        readonly length: number;
        readonly [n: number]: T;
    }
    ```

  - 只要是部署了 Iterator 接口的数据结构，都能将其转为数组。
  - 示例

    ```javascript
    // array-like
    let arrayLike = {
      0: '0',
      1: '1',
      2: '2',
      length: 3,
    };

    console.log(Array.from(arrayLike)); // 输出 [ '0', '1', '2' ]

    // iterable
    console.log(Array.from('hello')); // [ 'h', 'e', 'l', 'l', 'o' ]
    console.log(Array.from(new Set(['a', 'b']))); // [ 'a', 'b' ]
    ```

  - 可以接受第二个参数，作用类似于`map`函数

    ```javascript
    // 均输出 1 4 9
    console.log([1, 2, 3].map(x => x * x));
    console.log(Array.from([1, 2, 3], x => x * x));
    ```

  - 如果类似 map 的用法中需要`this`,可以通过`Array.from`的第三个参数传入。
  - 由于`Array.from`能够将字符串转为数组，且能够正确的处理各种 Unicode 字符，避免 JavaScript 将大于`\uFFFF`的 Unicode 字符算作两个字符的 BUG，我们可以用它来返回字符串的长度

    ```javascript
    function count(string) {
      return Array.from(string).length;
    }

    console.log(count('Hello')); // 输出 5
    console.log(count('𠮷')); // 输出 1
    console.log('𠮷'.length); // 输出 2, 这个不正确
    ```

- Array.of

  - Array 的缺陷，参数为 1 的时候与其它数量的表现也不同。
    ```javascript
    console.log(Array()); // []
    console.log(Array(3)); // [ <3 empty items> ], 实际是指定了数组的长度
    console.log(Array(3, 2)); // [ 3, 2 ]
    console.log(Array(3, 2, 1)); // [ 3, 2, 1 ]
    ```
  - Array.of 不存在参数不同而导致表现不同的行为。

    ```javascript
    console.log(Array.of()); // []
    console.log(Array.of(3)); // [3]
    console.log(Array.of(3, 2)); // [3, 2]
    ```

  - Array.of 基本上可以用来替代`Array()`和`new Array()`。

- copyWithin

  - 在当前数组复制，并返回当前数组
  - copyWithin(target: number, start: number, end: number)
  - target: 替换的起始位置。
  - start: 读取数据的起始位置(默认 0)，负值表示倒数。
  - end: 读取数据的终止位置(默认末尾)，负值表示倒数。

  ```javascript
  var a = Array.of(1, 2, 3, 4, 5);
  console.log(a); // [ 1, 2, 3, 4, 5 ]
  a.copyWithin(0, 2, 4);
  console.log(a); // [ 3, 4, 3, 4, 5 ]
  ```

- find, findIndex
  - 两个都用于找出符合条件的成员，一个返回成员，一个返回成员的位置。
  - 未找到符合条件的成员，find 返回 undefined, findIndex 返回-1。
  - 这两个方法第一个参数是一个回调函数，回调函数包括三个参数 `function(value, index, array)`, 分别是当前值，当前位置，原函数。
  - 这两个方法可以接受第二个参数，用于绑定回调函数中的`this`对象。
  ```javascript
  var l = [10, 8, 6, 4, 2];
  console.log(l.find(n => n <= 3)); // 2
  console.log(l.findIndex(n => n <= 3)); // 4
  ```
- fill

  - 用于填充一个数组，但是会抹去数组中原来的值

    ```javascript
    var l = [10, 8, 6, 4, 2];
    console.log(l.fill('a')); // 修改l后返回l, [ 'a', 'a', 'a', 'a', 'a' ]
    console.log(l); // [ 'a', 'a', 'a', 'a', 'a' ]
    ```

  - 可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置, 第三个参数默认到末尾

    ```javascrpit
    var l = [10, 8, 6, 4, 2];
    console.log(l.fill('b', 2, 3)); // [ 10, 8, 'b', 4, 2 ]
    console.log(l.fill('c', 2));    // [ 10, 8, 'c', 'c', 'c' ]
    ```

- entries, keys, values
  ```javascript
  var l = Array.of('a', 'b', 'c');
  for (let key of l.keys()) {
    console.log('key: ', key);
  }
  // values在我这报错：l.values is not a function
  for (let value of l.values()) {
    console.log('value: ', v);
  }
  for (let [key, value] of l.entries()) {
    console.log('key: ', key, ' => value: ', value);
  }
  ```
- includes
  - 用于检测数组是否包含某个给定的值。
  - ES2016 引入，不过我这里还不支持。
- 数组的空位
  - 在 ES5 中
    - `forEach()`, `filter()`, `every()`, `some()`都会跳过空位。
    - `map()`会跳过空位，但保留这个值。
    - `join()`, `toString()`会视空位为`undefined`, 而`undefined`和`null`会被处理为空字符串。
  - 在 ES6 中
    - 空位被视为`undefined`。

# 7 对象

- 简洁表示

  - 属性简写

    ```javascript
    function f(x, y) {
      return { x, y }; // 相当于 {x: x, y: y}
    }
    console.log(f(1, 2));
    ```

  - 函数简写

    ```javascript
    var Person = {
      name: 'Some one',
      hello() {
        // 相当于 hello: function() {
        return console.log('My name is', this.name);
      },
    };

    Person.hello();

    function getPoint() {
      var x = 1;
      var y = 2;
      return { x, y };
    }
    var p = getPoint();
    console.log(p.x, p.y);
    ```

  - 如果函数中有 Generator 函数，前面需要加上`*`
    ```
    var obj = {
        * m() {
            yield 'hello world';
        }
    }
    ```

- 属性名表达式

  - ES6 允许属性名放到大括号中

    ```javascript
    var lastWord = 'last world';

    var a = {
      'first world': 'hello',
      [lastWord]: 'world',
      ['h' + 'ello']: 'hello world',
    };

    console.log(a['first world']); // hello
    console.log(a[lastWord]); // world
    console.log(a['last world']); // world
    console.log(a['hello']); // hello world
    ```

  - 属性名表达式如果是对象，则会转为字符串`[object Object]`,如果内有多个对象做为属性名称，则会覆盖掉，只保留一个。

- name

  - 返回函数名，方法名。

    ```javascript
    function funcA() {}

    var b = {
      funcB() {},

      get age() {
        return 100;
      },

      set age(n) {},
    };

    console.log(funcA.name); // funcA
    console.log(b.funcB.name); // funcB

    const d = Object.getOwnPropertyDescriptor(b, 'age');
    console.log(d.get.name); // get age
    console.log(d.set.name); // set age
    ```

  - `bind`返回`bound`加上原函数名字。
  - `Function`函数返回`anonymous`。

    ```javascript
    console.log(new Function().name);
    ```

  - 如果对象的方法名是一个`Symbol`值，则`name`返回`Symbol`值的描述

    ```javascript
    const key = Symbol('hello key');
    var a = {
      [key]() {},
    };

    console.log(a[key].name);
    ```

- Object.is

  - 用于比较两个值是否相等，与`===`的行为基本一致。

  ```javascript
  // ES5
  console.log('foo' === 'foo'); // true
  console.log({} === {}); // false
  console.log(+0 === -0); // true
  console.log(NaN === NaN); // false
  // ES6
  console.log(Object.is('foo', 'foo')); // true
  console.log(Object.is({}, {})); // false
  console.log(Object.is(+0, -0)); // false
  console.log(Object.is(NaN, NaN)); // true
  ```

- Object.assign

  - 用于对象合并，将源对象所有可枚举属性复制到目标对象。
  - `assign(target: object, ...sources: any[])`。

    ```javascript
    var target = { a: 1 };
    var source1 = { b: 1 };
    var source2 = { c: 2 };

    Object.assign(target, source1, source2);
    console.log(target); // { a: 1, b: 1, c: 2 }
    ```

  - 由于`undefined`和`null`无法转为对象，所以如果他们作为首参数，就会报错。NaN 可以放在首参数。

    ```javascript
    Object.assign(undefined);
    Object.assign(null);
    ```

  - `undefined`, `null`, `NaN`，以及数值, 布尔值放入到非首参数的位置，都会被忽略。字符串包装的对象，才有可枚举属性(enumerable)。

    ```javascript
    var target = { a: 1 };

    var r = Object.assign(target, undefined, null, NaN, 10) === target;
    console.log(r); // true
    ```

  - 拷贝的属性是有限制的，只拷贝源对象的自身属性(非继承)和可枚举属性。

    ```javascript
    var target = {
      a: 1,
    };
    var source = Object.defineProperty(
      {
        b: 2,
      },
      'invisible',
      {
        enumerable: false,
        value: 'hello',
      }
    );
    Object.assign(target, source);
    console.log(target); // { a: 1, b: 2 }
    ```

  - 这是浅拷贝,如果源对象的某个属性是对象，则拷贝得到这个对象的引用。
  - 处理数组时，会把数组当成对象处理。

    ```javascript
    var a = ['a', 'b', 'c'];
    var b = ['d', 'e'];
    Object.assign(a, b);
    console.log(a); // [ d, e, c ]
    ```

    - 把数组视为属性名为 0，1，2 的对象。

# 8 Symbol

- 可以用于生成独一无二的属性名。

  ```javascript
  console.log(Symbol() === Symbol()); // false
  ```

- `Symbol`可以接受一个字符串做为参数，表示对`Symbol`实例的描述，主要为了帮助的区分。
  ```javascript
  // 以下输出 Symbol() Symbol() Symbol(hello) Symbol(NaN) Symbol(null) Symbol()
  console.log(
    Symbol(),
    Symbol(),
    Symbol('hello'),
    Symbol(NaN),
    Symbol(null),
    Symbol(undefined)
  );
  ```
  - 如果参数是对象，则会调用对象的`toString`方法，将其转为字符串。
- 可以转为字符串和布尔值，但是不能转为数值。

  ```javascript
  console.log(String(Symbol('hello'))); // Symbol(hello)
  console.log(Boolean(Symbol('world'))); // true
  console.log(Number(Symbol())); // 报错 Cannot convert a Symbol value to a number
  ```

- 做为属性名的应用

  ```javascript
  var mySymbol = Symbol();
  var a = {
    [mySymbol]: 'hello',
  };

  console.log(a[mySymbol]); // hello
  ```

  - 在对象的内部，使用`Symbol`定义属性时，`Symbol`值必须放在方括号中。否则属性名就变为字符串`mySymbol`了，而不是一个`Symbol`值。

- 定义常量的应用

  ```javascript
  var log = {
    levels: {
      DEBUG: Symbol('debug'),
      INFO: Symbol('info'),
      WARN: Symbol('warn'),
      FATAL: Symbol('fatal'),
    },
  };

  console.log(log.levels.DEBUG);
  ```

  - 常量使用`Symbol`值的最大好处就是不可能会有相同的值。

- 属性名的遍历

  - 使用`getOwnPropertySymbols`返回一个数组，成员为对象所有用作属性名的`Symbol`值。
  - 使用`Object.getOwnPropertyNames`得不到`Symbol`属性名。
  - `Reflect.ownKeys`可以得到所有类型的键名。

  ```javascript
  var b = Symbol('b');

  var o = {
    a: Symbol('a'),
    [b]: 'hello',
    c: 'world',
    foo() {},
  };

  var objectSymbols = Object.getOwnPropertySymbols(o);
  var objectNames = Object.getOwnPropertyNames(o);
  var allKey = Reflect.ownKeys(o);

  console.log('objectSymbols', objectSymbols); // objectSymbols [ Symbol(b) ]
  console.log('objectNames', objectNames); // objectNames [ 'a', 'c', 'foo' ]
  console.log('allKey', allKey); // allKey [ 'a', 'c', 'foo', Symbol(b) ]
  ```

- Symbol.for, Symbol.keyFor

  - `Symbol.for`与`Symbol`功能相似，不过前者会在全局登记，同一个参数返回的都是同一个`Symbol`值。
  - `Symbol.keyFor`，可以把登记过的`Symbol`值的参数找出来。

  ```javascript
  var s1 = Symbol.for('foo');
  var s2 = Symbol.for('foo');

  console.log(s1 === s2); // true
  console.log(Symbol.keyFor(s1)); // foo
  console.log(Symbol.keyFor(s2)); // foo
  ```

- Symbol.hasInstance

  - 指向一个内部方法，当其它对象调用`instanceof`运算符，判断是否是该对象的实例，就会调用该方法。

  ```javascript
  class MyClass {
    [Symbol.hasInstance](foo) {
      return foo instanceof Array;
    }
  }

  console.log([1, 2, 3] instanceof new MyClass()); // true

  class MyClass2 {
    [Symbol.hasInstance](value) {
      return value % 2 == 0;
    }
  }

  console.log(1 instanceof new MyClass2()); // false
  console.log(2 instanceof new MyClass2()); // true
  ```

- Symbol.isConcatSpreadable

  - 这是一个布尔值，表示该对象使用`Array.prototype.concat()`时，是否可以展开。
  - 数组的默认行为是可以展开的，`Symbol.isConcatSpreadable`为`true`或`undefined`时，都有这个效果。

    ```javascript
    let arr1 = ['c', 'd'];
    console.log(['a', 'b'].concat(arr1, ['e'])); // [ 'a', 'b', 'c', 'd', 'e' ]
    console.log(arr1[Symbol.isConcatSpreadable]); // undefined

    let arr2 = ['c', 'd'];
    arr2[Symbol.isConcatSpreadable] = false;

    // 以下输出 [ 'a', 'b', ['c', 'd', Symbol(Symbol.isConcatSpreadable)]: false], 'e' ]
    console.log(['a', 'b'].concat(arr2, ['e']));
    ```

  - 类似数组的对象这个效果是默认关闭的。

    ```javascript
    let obj = {
      0: 'c',
      1: 'd',
      length: 2,
    };

    console.log(['a', 'b'].concat(obj)); // [ 'a', 'b', { '0': 'c', '1': 'd', length: 2 } ]
    console.log(obj[Symbol.isConcatSpreadable]); // undefined

    obj[Symbol.isConcatSpreadable] = true;
    console.log(['a', 'b'].concat(obj)); // [ 'a', 'b', 'c', 'd' ]
    ```

- Symbol.species

  - 指向当前对象的构造函数，创建对象是，会调用这个方法。

    ```javascript
    class MyArray extends Array {
      static get [Symbol.species]() {
        console.log('hello');
        return Array;
      }
    }

    var l = new MyArray(1, 2, 3);

    console.log(l instanceof MyArray); // true
    console.log(l instanceof Array); // true

    var mapped = l.map(x => x * x); // hello

    console.log(mapped instanceof MyArray); //  false
    console.log(mapped instanceof Array); // true
    ```

  - 默认的`Symbol.species`相当于:

    ```javascript
    static get[Symbol.species]() {
        console.log('hello');
        return this;
    }
    ```

- Symbol.match

  - 指向一个函数，当执行`String.prototype.match()`时，如果该属性存在，就会调用它，并返回该方法的值。

    ```javascript
    class MyClass {
      [Symbol.match](string) {
        // return 'hello world'.indexOf(string);
        return 'hello world';
      }
    }

    console.log('e'.match(new MyClass())); // hello world
    ```

- Symbol.replace

  - 指向一个函数，当执行`String.prototype.replace()`时，如果该属性存在，就会调用它，并返回该方法的值。

    ```javascript
    class MyClass {
      [Symbol.replace](string) {
        return string + ' world';
      }
    }

    console.log('hello'.replace(new MyClass())); // hello world
    ```

- Symbol.replace

  - 指向一个函数，当执行`String.prototype.search()`时，如果该属性存在，就会调用它，并返回该方法的值。

    ```javascript
    class MyClass {
      [Symbol.search](string) {
        return 'hello world'.indexOf(string);
      }
    }

    console.log('l'.search(new MyClass())); // 2
    ```

- Symbol.split

  - 指向一个函数，当执行`String.prototype.split()`时，如果该属性存在，就会调用它，并返回该方法的值。

    ```javascript
    class MySplitter {
      constructor(value) {
        this.value = value;
      }

      [Symbol.split](string) {
        let index = string.indexOf(this.value);
        if (index === -1) {
          return string;
        }
        return [
          string.substr(0, index),
          string.substr(index + this.value.length),
        ];
      }
    }

    console.log('foobar'.split(new MySplitter('ba')));
    ```

- Symbol.iterator

  - 指向对象的默认遍历器方法。

    ```javascript
    var myIterable = {
      *[Symbol.iterator]() {
        yield 1;
        yield 2;
        yield 3;
      },
    };

    console.log([...myIterable]); // [1,2, 3]
    ```

  - `for...of`循环时，会调用`Symbol.iterator`

    ```javascript
    class MyCollection {
      constructor(...value) {
        this.value = value;
      }
      *[Symbol.iterator]() {
        let i = 0;
        while (this.value[i] !== undefined) {
          yield this.value[i];
          ++i;
        }
      }
    }

    let l = new MyCollection(1, 2, 3, 4);
    console.log(l); // MyCollection { value: [ 1, 2, 3, 4 ] }

    for (let value of l) {
      console.log(value); // 输出 1, 2, 3, 4
    }
    ```

- Symbol.toPrimitive

  - 指向一个方法，当对象被转为原始类型的值的时候，会调用这个方法，返回该对象对应的原始类型值。

    ```javascript
    let obj = {
      [Symbol.toPrimitive](value) {
        switch (value) {
          case 'number':
            return 123;
          case 'string':
            return 'string';
          case 'default':
            return 'default';
          default:
            throw new Error('unsupport');
        }
      },
    };

    console.log(2 * obj); // 246
    console.log(3 + obj); //default3
    console.log(3 - obj); //-120
    console.log(obj == 'default'); //true
    console.log(String(obj)); // string
    ```

- Symbol.toStringTag

  - 指向一个方法，当使用`Object.prototype.toString`时，会调用这个方法，并返回这个方法的值。

    ```javascript
    var obj = {
      get [Symbol.toStringTag]() {
        return 'hello world';
      },
    };

    console.log(String(obj));
    console.log(obj.toString());
    console.log(Object.prototype.toString.call(obj));
    ```

# 9 Set, Map

- Set

  - `Set`类似于数组，但没有重复的值。
    ```javascript
    let s = new Set();
    [1, 2, 3, 3, 2, 1].forEach(x => s.add(x));
    console.log(s); // Set { 1, 2, 3 }
    ```
  - `Set`可以接收数组做为参数。

    ```javascript
    let s = new Set([1, 2, 3, 3, 2, 1]);
    console.log(s); // Set { 1, 2, 3 }
    ```

  - `Set`属性:
    - `Set.prototype.constructor`: 构造函数。
    - `Set.prototype.size`: 返回成员总数。
  - `Set`操作方法:
    - `add(value)`: 添加某个值，返回`Set`结构本身。
    - `delete(value)`: 删除某个值，返回布尔值，表示是否删除成功。
    - `has(value)`: 返回布尔值，表示该值是否是`Set`的成员。
    - `clear()`: 清楚所有的成员，没有返回值。
  - `Set`遍历方法：

    - `keys`和`values`, 由于`Set`结构没有键名只有键值，所以二者行为一致。
    - `entries`, 返回键值对的遍历器。
    - `forEach`, 使用回调函数遍历每个成员。
    - 遍历的顺序即为插入的顺序。

    ```javascript
    let s = new Set([1, 3, 2]);

    console.log(s.keys()); // SetIterator { 1, 3, 2 }
    console.log(s.values()); // SetIterator { 1, 3, 2 }
    console.log(s.entries()); // SetIterator { [ 1, 1 ], [ 3, 3 ], [ 2, 2 ] }

    /**
     * 输出:
     * [ 1, 1 ]
     * [ 3, 3 ]
     * [ 2, 2 ]
     */
    for (let v of s.entries()) {
      console.log(v);
    }

    /**
     * 1 '=>' 1
     * 3 '=>' 3
     * 2 '=>' 2
     */
    s.forEach((k, v) => console.log(k, '=>', v));
    ```

  - `Array.of`和扩展运算符`...`:
    ```javascript
    let s = new Set(Array.of(1, 2, 3));
    console.log(s); // Set { 1, 2, 3 }
    let l = [...s];
    console.log(l); // [ 1, 2, 3 ]
    ```
  - 求并集，交集，差集

    ```javascript
    let a = new Set([1, 2, 3]);
    let b = new Set([1, 3, 4]);

    // 并集
    console.log('并集: ', new Set([...a, ...b])); // 并集:  Set { 1, 2, 3, 4 }

    // 交集
    console.log('交集: ', new Set([...a].filter(x => b.has(x)))); // 交集:  Set { 1, 3 }

    // 差集
    console.log('差集: ', new Set([...a].filter(x => !b.has(x)))); // 差集:  Set { 2 }
    ```

- WeakSet
  - `WeakSet`和`Set`类似，也是不重复的值的集合，但是有几个区别:
    - `WeakSet`的成员只能是对象，而不能是其它类型的值。
    - `WeakSet`是弱引用，当其它对象不再引用时，`WeakSet`里面的值会被回收，而不会考虑到对象是否还在`WeakSet`中。当外部对象消失的时候，`WeakSet`里面对应的引用也会消失，由于垃圾回收机制何时运行是不可预测的，因此 ES6 规定，`WeakSet`不可以遍历。
  - `WeakSet`可以接受数组或者类似数组的对象做为参数，实际上，具有`Iterable`接口的对象，都可以作为`WeakSet`的参数。
  - `WeakSet`由于弱引用的特性，使用`size`和`forEach`都会报错。
- Map

  - `Object`本质上是键值对的集合，只是键只能是使用字符串。
  - `Map`也是键值对的集合，但是键不限于字符串，各种类型的值(包括对象)都可以当键。
    ```javascript
    let l = [1, 2, 3];
    let m = new Map();
    m.set(l, 'it is l');
    console.log(m); // Map { [ 1, 2, 3 ] => 'it is l' }
    console.log(m.get(l)); // it is l
    ```
  - 只有对同一个对象的引用才会被`Map`视为同一个键。

    ```javascript
    let m = new Map();
    m.set({}, 'hello');
    console.log(m.get({})); // undefined
    ```

    - 前后的`{}`实际并不是同一个对象。

  - 另外，原书说`0`与`-0`, `undefined`, `null`, 也是两个不同的键，但是在我的运行环境里，却是被认为是同一个键。

    ```javascript
    let m = new Map();
    m.set(undefined, 'hello');
    console.log(m.get(undefined)); // hello

    m.set(null, 'hello');
    console.log(m.get(null)); // hello

    m.set(NaN, 'hello');
    console.log(m.get(NaN)); // hello

    m.set(0, 'hello');
    console.log(m.get(-0)); // hello
    ```

  - `Map`属性
    - `size`: 返回`Map`的成员总数。
  - `Map`操作方法：
    - `get(key)`: 获取`key`对应的值，如果找不到，则返回`undefined`。
    - `set(key, value)`: 设置`key`所对应的值，并返回`Map`本身。
    - `has(key)`: 返回一个布尔值，表示某个键是否在其中。
    - `delete(key)`: 删除某个键，返回一个布尔值表示成功与否。
    - `clear()`: 清理所有成员，没有返回值。
  - `Map`遍历方法：

    - `keys`: 返回键名的遍历器。
    - `values`: 返回键值的遍历器。
    - `entries`, 返回键值对的遍历器。
    - `forEach`, 使用回调函数遍历每个成员。
    - 遍历的顺序即为插入的顺序。

    ```javascript
    let m = new Map()
      .set(1, 'a')
      .set(2, 'b')
      .set(3, 'c');

    m.forEach((value, key, map) => console.log(value, key, map));
    ```

    - `forEach`还可以使用第二个参数，用来绑定`this`。

- WeakMap
  - `WeakMap`功能和`Map`类似，不同的地方在于:
    - `WeakMap`只接受对象做为键名。
  - `WeakMap`键名为弱引用，当外部对象消失后，其在`WeakMap`中的记录也会自动被移除。
  - `WeakMap`弱引用的只是键名，而不是键值。
  - `WeakMap`没有遍历的操作(没有`keys()`, `values()`, `entries()`),没有`size`属性,也没有`clear()`方法。

# 10 Proxy

- `Proxy`用于修改某些操作的默认行为，相当于对一些访问进行了拦截。

  ```javascript
  var proxy = new Proxy(
    {},
    {
      get: function(target, property) {
        return 35;
      },
    }
  );

  console.log(proxy.time, proxy.age); // 35 35
  ```

  - 拦截了对象属性的读取。

- `function get(target: object, propertyKey: PropertyKey, receiver?: any): any;`

  - 拦截对象属性的读取。

  ```javascript
  var person = {
    name: 'Abc',
  };

  var proxy = new Proxy(person, {
    get: function(target, property) {
      if (property in target) {
        return target[property];
      } else {
        throw new ReferenceError(
          'Property "' + property + '" does not exist. '
        );
      }
    },
  });

  console.log(proxy.name); // Abc
  console.log(proxy.age); // ReferenceError: Property "age" does not exist.
  ```

  - 以上代码中，如果没有拦截，则会返回`undefined`。

  ```javascript
  function createArray(...elements) {
    let handler = {
      get(target, propertyKey, receiver) {
        let index = Number(propertyKey);
        if (index < 0) {
          propertyKey = String(target.length + index);
        }

        return Reflect.get(target, propertyKey, receiver);
      },
    };

    let target = [];
    target.push(...elements);
    return new Proxy(target, handler);
  }

  var arr = createArray('a', 'b', 'c');
  console.log(arr[-1]); // c
  ```

  - 以上代码中，使用`get`拦截，实现了数组读取负数的索引。

  ```javascript
  const target = Object.defineProperties(
    {},
    {
      foo: {
        value: 123,
        writable: false,
        configurable: false,
      },
    }
  );

  const proxy = new Proxy(target, {
    get: function(target, propertyKey) {
      return 'Abc';
    },
  });

  console.log(proxy.name); // Abc
  console.log(target.foo); // 123
  console.log(proxy.foo); // 报错
  ```

  - 如果对象的属性不可配置且不可写，则该属性不能被代理，通过代理访问会报错。

- `function set(target: object, propertyKey: PropertyKey, value: any, receiver?: any): boolean;`

  - 拦截某个属性的赋值操作

  ```javascript
  let handler = {
    set: function(target, propertyKey, value) {
      if (propertyKey === 'age') {
        if (!Number.isInteger(value)) {
          throw new TypeError('The age is not an integer');
        }
        if (value < 0 || value > 200) {
          throw new RangeError('The age seems invalid');
        }
      }

      target[propertyKey] = value;

      console.log('Property "' + propertyKey + '" is ' + value);
    },
  };

  var person = new Proxy({}, handler);

  person.age = 100; // Property "age" is 100
  // person.age = '100'; // TypeError: The age is not an integer
  // person.age = 'hello'; // TypeError: The age is not an integer
  // person.age = 201; // RangeError: The age seems invalid
  ```

  - 以上代码拦截了`set`操作，对`age`属性做了判断。在严格模式下，以上代码会报错`TypeError: 'set' on proxy: trap returned falsish for property 'age'`。

  ```javascript
  function invariant(key, action) {
    if (key[0] === '_') {
      throw new Error(`Invalid attemp to ${action} private "${key}"`);
    }
  }

  let handler = {
    get: function(target, propertyKey) {
      invariant(propertyKey, 'get');
      return target[propertyKey];
    },
    set: function(target, propertyKey, value) {
      invariant(propertyKey, 'set');
      target[propertyKey] = value;
    },
  };

  var proxy = new Proxy({}, handler);

  console.log(proxy.age); // undefined
  console.log(proxy._age); // Error: Invalid attemp to get private "_age"
  ```

  - 假设我们认为以下划线开头是内部属性，不应该被外部使用。以上代码只要读写操作的属性第一个字符是下划线，就会报错，从而达到禁止读写内部属性的目的。

- `function apply(target: Function, thisArgument: any, argumentsList: ArrayLike<any>): any;`

  - 拦截**函数调用**、`call`和`apply`

  ```javascript
  let target = () => {
    return 'I am the target .';
  };
  let handler = {
    apply: function() {
      return 'I am the proxy. ';
    },
  };

  let p = new Proxy(target, handler);
  console.log(p()); // I am the proxy.
  ```

  - 以上代码演示了函数调用的拦截。

  ```javascript
  var twice = {
    apply: function(target, ctx, args) {
      return Reflect.apply(...arguments) * 2;
    },
  };

  function sum(a, b) {
    return a + b;
  }

  var proxy = new Proxy(sum, twice);

  console.log(proxy(1, 2)); // 6 拦截函数调用
  console.log(proxy.call(null, 3, 4)); // 14 拦截call
  console.log(proxy.apply(null, [5, 6])); // 22 拦截apply
  ```

  - 关于`call`和`apply`的更多信息，请参考[Javascript 中的 this,call,apply,bind 函数](http://alex-my.xyz/js-thi-call-apply-bind)

- `function has(target: object, propertyKey: PropertyKey): boolean;`

  - 拦截`HasProperty`操作，即判断对象是否具有某个属性的时候，这个方法就会生效。

  ```javascript
  var handler = {
    has(target, propertyKey) {
      if (propertyKey[0] === '_') {
        return false;
      }
      return propertyKey in target;
    },
  };

  var target = {
    _prop: '_foo',
    prop: 'foo',
  };

  var proxy = new Proxy(target, handler);

  console.log('_prop' in proxy, 'prop' in proxy); // false true
  ```

  - 以上代码使用`has`拦截了某些属性，不被`in`运算符发现。

  ```javascript
  Object.preventExtensions(target);
  ```

  - 如果对象不可配置或者禁止扩展，这时候`has`拦截会报错。
  - `has`对`for ... in`不生效。

- `function construct(target: Function, argumentsList: ArrayLike<any>, newTarget?: any): any;`

  - 拦截`new`命令。
  - `construct`返回必须是一个对象，否则报错。

  ```javascript
  var handler = {
    construct(target, args, newTarget) {
      console.log(...args);
      return new target(...args);
    },
  };

  var proxy = new Proxy(function() {}, handler);

  new proxy(123, 456).name;
  ```

- `function deleteProperty(target: object, propertyKey: PropertyKey): boolean;`

  - 拦截`delete`操作，当返回`false`或者报错的时候，当前属性就无法被`delete`删除。

  ```javascript
  var handler = {
    deleteProperty(target, propertyKey) {
      if (propertyKey[0] === '_') {
        throw new Error(`Invalid attemp to delete private ${propertyKey}`);
      }
      return true;
    },
  };

  var target = {
    _prop: 'foo',
  };

  var proxy = new Proxy(target, handler);

  delete proxy._prop; // Error: Invalid attemp to delete private _prop
  ```

- `function defineProperty(target: object, propertyKey: PropertyKey, attributes: PropertyDescriptor): boolean;`

  - 拦截`Object.defineProperty`操作，如果返回`false`或者报错的时候，就无法添加新属性。

  ```javascript
  var handler = {
    defineProperty(target, propertyKey, attributes) {
      if (propertyKey[0] === '_') {
        return false;
      }
      return true;
    },
  };

  var target = {
    name: 'target name',
  };

  var proxy = new Proxy(target, handler);
  proxy.foo = 'foo';
  proxy._foo = '_foo'; // TypeError: 'defineProperty' on proxy: trap returned falsish for property '_foo'
  ```

- `function getOwnPropertyDescriptor(target: object, propertyKey: PropertyKey): PropertyDescriptor;`

  - 拦截`Object.getOwnPropertyDescriptor()`

  ```javascript
  var handler = {
    getOwnPropertyDescriptor(target, propertyKey) {
      if (propertyKey[0] === '_') {
        return;
      }
      return Object.getOwnPropertyDescriptor(target, propertyKey);
    },
  };
  var target = {
    _foo: '_foo',
    bar: 'bar',
  };

  var proxy = new Proxy(target, handler);

  console.log(Object.getOwnPropertyDescriptor(proxy, '_foo')); // undefined
  console.log(Object.getOwnPropertyDescriptor(proxy, 'bar')); // { value: 'bar', writable: true, enumerable: true, configurable: true }
  ```

  - 以上代码中禁止获取第一个字符为下划线的属性的信息，返回`undefined`。

- `function getPrototypeOf(target: object): object;`
  - 拦截获取对象原型的方法。
    - `Object.prototype.__proto__`
    - `Object.prototype.isPrototypeOf()`
    - `Object.getPrototypeOf()`
    - `Reflect.getPrototypeOf()`
    - `instanceof`
- `isExtensible? (target: T): boolean;`

  - 拦截`Object.isExtensible`方法。
  - 其返回值必须与目标对象的`isExtensible`属性保持一致，否则会报错。

  ```javascript
  var target = {};

  var proxy = new Proxy(target, {
    isExtensible: function(target) {
      console.log('called');
      return true;
    },
  });

  Object.isExtensible(proxy); // called
  console.log(Object.isExtensible(proxy) === Object.isExtensible(target)); // true
  ```

  - 在以上示例中，如果`isExtensible`返回的`false`，与`target`的实际情况不同，就会报错。

- `ownKeys? (target: T): PropertyKey[];`

  - 拦截以下操作:
    - `Object.getOwnPropertyNames()`
    - `Object.getOwnPropertySymbols()`
    - `Object.keys()`

  ```javascript
  var target = {
    _bar: '_bar',
    _prop: '_prop',
    name: 'name',
  };

  var handler = {
    ownKeys(target) {
      return Reflect.ownKeys(target).filter(key => key[0] !== '_');
    },
  };

  var proxy = new Proxy(target, handler);

  console.log(Object.keys(target)); // [ '_bar', '_prop', 'name' ]
  console.log(Object.keys(proxy)); // [ 'name' ]
  ```

  - 以上代码中过滤了第一个字符为下划线的属性名。
  - 实际上，有三类属性会被`ownKeys`过滤，写出来也不会出现。
    - 目标对象上不存在的属性
    - 属性名为`Symbol`值
    - 不可遍历的属性
  - 具体看以下示例:

  ```javascript
  var target = {
    a: 1,
    b: 2,
    [Symbol.for('secret')]: 3,
  };

  Object.defineProperty(target, 'key', {
    enumerable: false,
  });

  var handler = {
    ownKeys(target) {
      return ['a', 'd', Symbol.for('secret'), 'key'];
    },
  };

  var proxy = new Proxy(target, handler);

  console.log(Object.keys(proxy)); // [ 'a' ]
  ```

- `preventExtensions? (target: T): boolean;`

  - 拦截`Object.preventExtensions()`。
  - 该方法有一个限制，只有目标对象不可扩展时，`preventExtensions`才能返回 true，否则会报错。
  - 为了解决这个问题，可以在`preventExtensions`中把`target`也修改下。

  ```javascript
  var target = {};

  var handler = {
    preventExtensions(target) {
      Object.preventExtensions(target);
      return true;
    },
  };

  var proxy = new Proxy(target, handler);

  Object.preventExtensions(proxy);
  ```

- `setPrototypeOf? (target: T, v: any): boolean;`

  - 拦截`Object.setPrototypeOf`方法。
  - 如果对象不可扩展，则`setPrototypeOf`不得改变目标对象的原型。

- `Proxy.revocable`

  - 返回一个可取消的实例，当执行`revoke`函数后，`proxy`就不可再访问了。

  ```javascript
  var target = {};

  var handler = {};

  var { proxy, revoke } = Proxy.revocable(target, handler);

  console.log(proxy.name); // undefined

  revoke();

  console.log(proxy.name); // TypeError: Cannot perform 'get' on a proxy that has been revoked
  ```

- this 问题

  - 在代理的情况下，目标对象内部的`this`会指向`Proxy`代理。

  ```javascript
  var target = {
    foo() {
      console.log(this === proxy);
    },
  };

  var proxy = new Proxy(target, {});

  target.foo(); // false
  proxy.foo(); // true this指向了proxy
  ```

  - 有些原生对象的内部属性，只有通过对象的`this`才能取到，所以`Proxy`无法代理这些原生对象的属性。

  ```javascript
  var target = new Date();
  var proxy = new Proxy(target, {});

  console.log(proxy.getDate()); // TypeError: this is not a Date object.
  ```

  - 这个时候需要把`this`指针绑定原始对象，才可以解决这个问题。

  ```javascript
  var target = new Date();

  var handler = {
    get(target, propertyKey) {
      if (propertyKey === 'getDate') {
        return target.getDate.bind(target);
      }
      return Reflect.get(target, propertyKey);
    },
  };

  var proxy = new Proxy(target, handler);

  console.log(proxy.getDate()); // 25
  ```

# 11 Reflect

- `Reflect`对象和`Proxy`对象一样，也是 ES6 为操作对象而提供的新 API。

  - 将`Object`对象的一些明显属于语言内部的方法，放到`Reflect`上。目前是同时在`Object`和`Reflect`上部署，而未来只部署在`Reflect`上。

    ```javascript
    var obj = {};
    console.log(Object.isExtensible(obj)); // true
    console.log(Reflect.isExtensible(obj)); // true
    ```

  - 修改某些`Object`方法的返回结果。例如`Object.defineProperty()`在无法定义属性时，会抛出错误。而`Reflect.defineProperty()`则会返回`false`。
  - 让`Object`操作变成函数行为。

    ```javascript
    var obj = {};
    console.log('foo' in obj); // false
    console.log(Reflect.has(obj, 'foo')); // false
    ```

  - `Reflect`对象的方法与`Proxy`对象的方法一一对应。使用`Reflect`对应的方法，可以完成默认的行为。

    ```javascript
    var target = {};
    var handler = {
      get(target, propertyKey) {
        if (propertyKey[0] === '_') {
          throw new Error(`Invalid attamp to get param ${propertyKey}`);
        }
        // 完成默认的行为
        return Reflect.get(target, propertyKey);
      },
    };

    var proxy = new Proxy(target, handler);

    console.log(proxy.name); // undefined
    console.log(proxy._name); // Error: Invalid attamp to get param _name
    ```

- `Reflect`对象一共有 13 个静态方法：

  ```javascript
  function apply(target: Function, thisArgument: any, argumentsList: ArrayLike<any>): any;
  function construct(target: Function, argumentsList: ArrayLike<any>, newTarget?: any): any;
  function defineProperty(target: object, propertyKey: PropertyKey, attributes: PropertyDescriptor): boolean;
  function deleteProperty(target: object, propertyKey: PropertyKey): boolean;
  function get(target: object, propertyKey: PropertyKey, receiver?: any): any;
  function getOwnPropertyDescriptor(target: object, propertyKey: PropertyKey): PropertyDescriptor;
  function getPrototypeOf(target: object): object;
  function has(target: object, propertyKey: PropertyKey): boolean;
  function isExtensible(target: object): boolean;
  function ownKeys(target: object): Array<PropertyKey>;
  function preventExtensions(target: object): boolean;
  function set(target: object, propertyKey: PropertyKey, value: any, receiver?: any): boolean;
  function setPrototypeOf(target: object, proto: any): boolean;
  ```

# 12 Promise

- `Promise`是异步编程的一种解决方案。
- `Promise`有两个特点：
  1. 对象的状态不受外界影响。`Promise`对象一共有三种状态: `Pending`(进行中)、`Fulfilled`(已成功)、`Rejected`(已失败)。只有异步操作的结果可以决定当前是处于什么状态，任何其他操作都无法改变这个状态。
  2. 一旦状态发生改变，就不会在变化。有且只有两种变化从`Pending`变为`Fulfilled`，从`Pending`变为`Rejected`。
- `Promise`的缺点
  1. 无法取消`Promise`，一旦建立就会立即执行，无法中途中断。
  2. 如果没有设置回调函数，`Promise`内部抛出错误，不会反应到外部。
  3. 当处于`Pending`状态的时候，无法得知目前进展到哪一个阶段(是刚开始还是将要结束了)。
- 基本用法

  - 创建一个`Promise`实例

    ```javascript
    var promise = new Promise(function(resolve, reject) {
      // 如果成功, 则调用resolve函数; 失败则调用reject函数
    });
    ```

  - 指定成功，失败状态的回调函数

    ```javascript
    promise.then(
      function(value) {
        // promise执行resolve时调用
      },
      function(value) {
        // rpromise执行reject时调用
      }
    );
    ```

    - `then`方法可以接受两个回调函数做为参数。其中，第二个是可选的。这两个回调函数都接受`Promise`对象传出的值做为参数。

  - 示例:

    ```javascript
    function timeout(ms) {
      return new Promise((resolve, reject) => {
        // 为了测试方便, 用ms的大小做为成功和失败的区别
        if (ms > 2000) {
          // setTimeout(resolve, ms, 'Resolve done.');
          resolve('Resolve done.');
        } else {
          // setTimeout(reject, ms, 'Reject done.');
          reject('Resolve done.');
        }
      });
    }

    timeout(2000).then(
      value => {
        console.log('Success: ', value);
      },
      value => {
        console.log('Failed: ', value);
      }
    ); // 输出: Failed:  Resolve done.
    ```

- 执行顺序

  ```javascript
  let promise = new Promise((resolve, reject) => {
    console.log('before resolve');
    resolve();
    console.log('after resolve');
  });

  promise.then(value => {
    console.log('promise then');
  });

  console.log('hello world');

  // 输出:
  // before resolve
  // after resolve
  // hello world
  // promise then
  ```

  - `Promise`建立后立即执行，所以会先输出`before resolve`和`after resolve`。
  - `then`指定的回调函数，需要在当前脚本所有函数都执行完之后才会执行，所以会输出`hello world`。
  - 最后执行回调函数，输出`promise then`。

- 一个异步的操作结果是返回另一个异步操作

  ```javascript
  var p1 = new Promise((resolve, reject) => {
    setTimeout(() => resolve('p1 done'), 10000);
  });

  var p2 = new Promise((resolve, reject) => {
    setTimeout(() => resolve(p1), 1000);
  });

  p2.then(result => console.log(result)).catch(error => console.log(error));
  ```

  - `p2`需要等待`p1`结束后才会执行。

- `Promise.prototype.then()`

  - 作用是为`Promise`添加状态发生改变时候触发的回调函数，可以添加两个函数。
  - 第一个是`resolve`(成功)时触发的。
  - 第二个是`reject`(失败)时触发的，这个可选。
  - `then`返回的是一个新的`Promise`实例，因此可以采用链式写法。

  ```javascript
  var p1 = new Promise((resolve, reject) => {
    resolve();
  });

  p1.then(() => {
    console.log('first then');
    return 'first then end';
  }).then(value => {
    console.log('second then', value);
  });

  // 输出
  // first then
  // second then first then end
  ```

  - 在回调函数中，也可以直接返回一个`Promise`实例。

  ```javascript
  var p1 = new Promise((resolve, reject) => {
    resolve();
  });

  p1.then(() => {
    console.log('first then');

    return new Promise((resolve, reject) => {
      setTimeout(() => resolve('first setTimeout end'), 5000);
    });
  }).then(value => {
    console.log('second then', value);
  });

  // 输出
  // first then
  // second then first setTimeout end
  ```

- `Promise.prototype.catch()`

  - 用于指定发生错误时的回调函数。

  ```javascript
  var promise = new Promise((resolve, reject) => {
    reject(new Error('Promise reject.'));
  });

  promise
    .then(
      () => {},
      () => {
        console.log('then reject');
      }
    )
    .catch(() => {
      console.log('catch');
    }); // 输出 then reject
  ```

  - 如果已经在`then`中指定了`reject`的回调函数，则不会触发`catch`。即使`Promise`中抛出异常，也会被`then`中的`reject`获取。

  ```javascript
  var promise = new Promise((resolve, reject) => {
    reject('hello reject');
  });

  promise
    .then(() => {})
    .catch(() => {
      console.log('catch');
    }); // 输出 catch
  ```

  - 如果没有在`then`中指定`reject`的回调函数，就会被`catch`获取。

  ```javascript
  var promise = new Promise((resolve, reject) => {
    resolve('Promise resolve');
    throw new Error('Promise Error');
  });

  promise
    .then(
      () => {
        console.log('then resolve');
      },
      () => {
        console.log('then reject');
      }
    )
    .catch(() => {
      console.log('catch');
    }); // 输出 then resolve
  ```

  - 如果已经变成了`resolve`,那么再抛出错误也是无效的。

  ```javascript
  promise
    .then(() => {})
    .then(() => {})
    .catch(() => {});
  ```

  - `Promise`的错误会一直传递，直到被捕获位置，上面的代码中有三个`Promise`实例，任何一个抛出错误，都会被`catch`捕获。
  - 一般来说，尽量不要在`then`中定义`reject`的回调函数，而是使用`catch`方法。
  - 如果没有指定捕获异常，`Promise`抛出的错误是不会传递到外层代码的。

  ```javascript
  var promise = new Promise((resolve, reject) => {
    resolve('hello');
  });

  promise
    .then(() => {
      x + 2;
    })
    .catch(() => {
      y + 2;
    })
    .catch(() => {
      console.log('last catch');
    });
  ```

  - `catch`中还可以再抛出错误，这个错误可以被之后的`catch`捕获到。

- `Promise.all(iterable)`

  - iterable : An iterable object such as an Array or String.
  - This method can be useful for aggregating the results of multiple promises.
  - return value:
    - Fulfillment:
      1. If an empty iterable is passed, then this method returns (synchronously) an already resolved promise.
      2. If all of the passed-in promises fulfill, or are not promises, the promise returned by Promise.all is fulfilled asynchronously.
      3. In all cases, the returned promise is fulfilled with an array containing all the values of the iterable passed as argument (also non-promise values).
    - Rejection:
      1. If any of the passed-in promises reject, Promise.all asynchronously rejects with the value of the promise that rejected, whether or not the other promises have resolved.
  - examples:

    - `Promise.all` waits for all fulfillments (or the first rejection).

      ```javascript
      // ----------------- all fulfillments
      var p1 = 1337;
      var p2 = Promise.resolve(3);
      var p3 = new Promise((resolve, reject) => {
        setTimeout(resolve, 3000, 'p3 resolve');
      });
      Promise.all([p1, p2, p3]).then(values => {
        console.log(values);
      });
      // [ 1337, 3, 'p3 resolve' ]
      // ----------------- return first rejection
      var p1 = 1337;
      var p2 = Promise.reject(3);
      var p3 = new Promise((resolve, reject) => {
        setTimeout(reject, 3000, 'p3 reject');
      });
      Promise.all([p1, p2, p3])
        .then(values => {
          console.log('Promise all then ', values);
        })
        .catch(error => {
          console.log('Promise all catch ', error);
        });
      // Promise all catch  3
      ```

  - Promise.all is rejected if any of the elements are rejected. For example, if you pass in four promises that resolve after a timeout and one promise that rejects immediately, then Promise.all will reject immediately.

- `Promise.race(iterable)`

  - iterable: An iterable object, such as an Array.
  - The race function returns a Promise, adopting that first promise's value as its value.
  - return value:
    - If the iterable passed is empty, the promise returned will be forever pending.
    - If the iterable contains one or more non-promise value and/or an already resolved/rejected promise, then Promise.race will resolve to the first of these values found in the iterable.
  - examples:

    ```javascript
    // ----------------- the promise returned will be forever pending
    var promise = Promise.race([]);

    promise
      .then(value => {
        console.log('Promise race then');
      })
      .catch(error => {
        console.log('Promise race catch');
      });

    console.log(promise); // Promise { <pending> }
    ```

    ```javascript
    // ----------------- then Promise.race will resolve to the first of these values found in the iterable.
    var p1 = new Promise((resolve, reject) => {
      setTimeout(resolve, 400, 'p1 resolve');
    });

    var p2 = new Promise((resolve, reject) => {
      setTimeout(reject, 300, 'p2 reject');
    });

    var p3 = new Promise((resolve, reject) => {
      setTimeout(resolve, 200, 'p3 resolve');
    });

    var promise = Promise.race([p1, p2, p3]);

    promise
      .then(value => {
        console.log('Promise race then ', value);
      })
      .catch(error => {
        console.log('Promise race catch', error);
      });

    console.log(promise);
    // output:
    // Promise { <pending> }
    // Promise race then  p3 resolve
    ```

- `Promise.resolve()`

  - `Promise.resolve(value)`
  - `Promise.resolve(promise)`
  - `Promise.resolve(thenable)`: a object has a then method
  - return value: A Promise that is resolved with the given value, or the promise passed as value, if the value was a promise object.
  - examples:

    - `Promise.resolve(value)`

      ```javascript
      var p1 = Promise.resolve('hello').then(value =>
        console.log('then ', value)
      ); // then  hello
      var p2 = Promise.resolve([1, 2, 3]).then(value =>
        console.log('then ', value)
      ); // then  [ 1, 2, 3 ]
      ```

    - `Promise.resolve(promise)`

      ```javascript
      // ----------------- return the param as result
      var p1 = new Promise((resolve, reject) => {
        resolve('hello p1');
      });

      var p2 = Promise.resolve(p1);

      console.log(p1 === p2); // true

      p2.then(value => console.log(value)); // hello p1
      ```

    - `Promise.resolve(thenable)`

      ```javascript
      var obj = {
        then(resolve, reject) {
          resolve('hello, obj resolve');
        },
      };

      var promise = Promise.resolve(obj);

      console.log(obj === promise); // false

      promise.then(value => console.log(value)); // hello, obj resolve
      ```

- `Promise.reject(reason)`
  - A Promise that is rejected with the given reason.

# 13 Iterable & Iterator

- In order to be iterable, an object must implement the @@iterator method.
- An object is an iterator when it implements a next() method (A zero arguments function that returns an object with two properties: `done`, `value`).
  - done(boolean):
    - Has the value true if the iterator is past the end of the iterated sequence.
    - Has the value false if the iterator was able to produce the next value in the sequence.
  - value:
    - any JavaScript value returned by the iterator. Can be omitted when done is true.
- `String`, `Array`, `TypedArray`, `Map` and `Set` are all built-in iterables, because each of their prototype objects implements an @@iterator method.
- Iterable examples:

  - User-defined iterables:

    ```javascript
    var iter = {};

    iter[Symbol.iterator] = function*() {
      yield 1;
      yield 2;
      yield 3;
    };

    for (let v of iter) {
      console.log(v);
    }
    ```

  - If an iterable's @@iterator method doesn't return an iterator object, it is likely to result in runtime exceptions or buggy behavior

    ```javascript
    var iter = {};

    iter[Symbol.iterator] = () => 1;

    for (let v of iter) {
      console.log(v);
    }

    // TypeError: Result of the Symbol.iterator method is not an object
    ```

- Iterator examples:

  - Simple iterator

    ```javascript
    function makeIterator(array) {
      var index = 0;

      return {
        next: function() {
          return index < array.length
            ? {
                value: array[index++],
                done: false,
              }
            : {
                done: true,
              };
        },
      };
    }

    var iter = makeIterator([1, 2]);

    console.log(iter.next()); // { value: 1, done: false }
    console.log(iter.next()); // { value: 2, done: false }
    console.log(iter.next()); // { done: true }
    ```

  - With ES6 class

    ```javascript
    class MyClass {
      constructor(data) {
        this.index = 0;
        this.data = data;
      }

      [Symbol.iterator]() {
        return {
          next: () => {
            if (this.index < this.data.length) {
              return {
                value: this.data[this.index++],
                done: false,
              };
            } else {
              return {
                done: true,
              };
            }
          },
        };
      }
    }

    var l = new MyClass([1, 2, 3, 4]);

    for (let o of l) {
      console.log(o);
    }
    ```

# 14 Generator

- `Generator` function return a generator object which conforms to both the iterable protocol and the iterator protocol.
- Methods:
  - next: returns a value yielded by the `yield` expression.
  - return: returns the given value and finishes the generator.
  - throw: throws an error to a generator.
- Examples:

  ```javascript
  function* gen() {
    yield 'hello';
    yield 'world';
    return 'ending';
  }

  var g = gen();

  console.log(g.next()); // { value: 'hello', done: false }
  console.log(g.next()); // { value: 'world', done: false }
  console.log(g.next()); // { value: 'ending', done: true }
  console.log(g.next()); // { value: undefined, done: true }
  ```

- `next(value)`

  - The value to send to the generator.

  ```javascript
  function* gen(x) {
    let y = yield x + 1;
    let z = yield y * 2;
    return x + y + z;
  }

  var g = gen(1);
  console.log(g.next()); // { value: 2, done: false }, x = 1, return valus is 2
  console.log(g.next(10)); // { value: 6, done: false }, yield(x + 1) = 10, so y = 10, and return value is 20
  console.log(g.next(100)); // { value: 8, done: true }, yield(y * 2) = 100, and x = 1, y = 10, so return value is 111
  ```

  - **`x` is treated as the last yield value**.

- `return(value)`

  - The value to return.
  - If return(value) is called on a generator that is already in "completed" state, the generator will remain in "completed" state.

    ```javascript
    function* gen(x) {
      yield 1;
      yield 2;
      yield 3;
    }

    var g = gen();
    console.log(g.next()); // { value: 1, done: false }
    console.log(g.return('hello')); // { value: 'hello', done: true } it is over.
    console.log(g.next()); // { value: undefined, done: true }
    ```

  - If try ... finally in generator function.

    ```javascript
    function* gen(x) {
      try {
        yield 1;
        yield 2;
      } finally {
        yield 3;
        yield 4;
      }
      yield 5;
    }

    var g = gen();
    console.log(g.next()); // { value: 1, done: false }
    console.log(g.return('hello')); // { value: 3, done: false }, finally is called first
    console.log(g.next()); // { value: 4, done: false }
    console.log(g.next()); // { value: 'hello', done: true }, return 'hello' after finally
    console.log(g.next()); // { value: undefined, done: true }, generator function is over
    console.log(g.next()); // { value: undefined, done: true }, generator function is over
    ```

- `yield*`

  - The **yield\*** expression is used to delegate to another generator or iterable object.
  - syntax: `yield* [[expression]]`
  - expression: which returns an iterable object.
  - examples:

    ```javascript
    function* gen1() {
      yield 1;
      yield 2;
    }

    function* gen2() {
      yield 'a';
      yield* gen1();
      yield 'b';
    }

    var g = gen2();

    for (let v of g) {
      console.log(v);
    }

    // output: a 1 2 b
    ```

    ```javascript
    function* gen() {
      yield* 'hello';
      yield* [1, 2, 3];
    }

    var g = gen();

    for (let value of g) {
      console.log(value);
    }

    // output: h e l l o 1 2 3
    ```

# 15 async & await

- `async`和`await`看上去就像是简化`Promise`或`Generator`操作。
- 一个简单的异步示例, 每个 5 秒输出一些文字，有多个这样的异步操作

  - `Promise`版本

    ```javascript
    var promise1 = new Promise((resolve, reject) => {
      setTimeout(resolve, 5000, 'promise1 timeout');
    });

    var promise2 = new Promise((resolve, reject) => {
      setTimeout(resolve, 5000, 'promise2 timeout');
    });

    promise1
      .then(value => {
        console.log('promise1 then', value);
        return promise2;
      })
      .then(value => {
        console.log('promise2 then', value);
      })
      .catch(error => {
        console.log('promise catch', error);
      });

    // 输出:
    // promise1 then promise1 timeout
    // promise2 then promise2 timeout
    ```

    - 该版本对于多个`Promise`串行执行的情况，需要在`then`里执行下一个的`Promise`实例，看上去就是一层层的嵌套。

  - `Generator`版本

    ```javascript
    function timeout(value) {
      setTimeout(() => console.log(value), 5000);
    }

    var gen = function*() {
      yield timeout('timeout 1');
      yield timeout('timeout 2');
    };

    var g = gen();
    g.next();
    g.next();

    // 输出
    // timeout 1
    // timeout 2
    ```

    - 该版本比`Promise`版本好看的多，但如果不借助`co`模块或类似的辅助，对于多个异步，我们需要写出多个`next`函数。

  - `async & await`版本

    ```javascript
    function timeout(value, ms) {
      return new Promise((resolve, reject) => {
        setTimeout(resolve, ms, value);
      });
    }

    async function foo() {
      let x = await timeout('hello timeout 1', 5000);
      let y = await timeout('hello timeout 2', 5000);
      return [x, y];
    }

    foo().then(v => console.log(v));

    // 输出: [ 'hello timeout 1', 'hello timeout 2' ]
    ```

    - 该版本的异步看上去就和同步的写法差不多，就是多了`async`和`await`，简单明了。

- `async`函数最终返回一个`Promise`示例，所以，后面可以接`then`, `catch`等。
- `await`命令后是一个`Promise`对象，如果不是，会被转为`Promise`对象。
- `await`命令只能在`async`函数中使用。
- `async`函数返回的`Promise`实例，需要等到内部所有的`await`命令后的`Promise`实例执行完，才会发生状态改变。一旦其中一个`await`命令后的`Promise`实例抛出错误或者`reject`，`async`函数返回的`Promise`实例就会立即改变状态，且后续的`await`命令就会中断执行。
- 为了防止抛出错误中断执行，可以使用`try...catch`，或者是`await`命令后的`Promise`实例接上`catch`。

  ```javascript
  // -------------------- try ... catch
  function timeout(value, ms) {
    return new Promise((resolve, reject) => {
      if (value === 'hello timeout 1') {
        reject('reject 1'); // 第一个直接reject
      } else {
        setTimeout(resolve, ms, value);
      }
    });
  }

  async function foo() {
    try {
      let x = await timeout('hello timeout 1', 5000);
    } catch (error) {
      console.log('time 1 catch,', error);
    }
    let y = await timeout('hello timeout 2', 5000);

    return 'foo end';
  }

  let pre = new Date().getTime();

  foo()
    .then(v => {
      console.log(v);

      let end = new Date().getTime();
      console.log('time: ', end - pre);
    })
    .catch(error => console.log(error));

  // 输出:
  // time 1 catch, reject 1
  // foo end
  // time:  5026

  // -------------------- Promise.catch
  async function foo() {
    await timeout('hello timeout 1', 5000).catch(error =>
      console.log('error: ', error)
    );
    let y = await timeout('hello timeout 2', 5000);

    return 'foo end';
  }
  ```

- 在上面的`async & await`版本中，后一个异步操作要等到前一个异步操作执行完后才会执行，如果希望同时执行，可以修改为以下:

  ```javascript
  function timeout(value, ms) {
    return new Promise((resolve, reject) => {
      setTimeout(resolve, ms, value);
    });
  }

  async function foo() {
    let x = timeout('hello timeout 1', 5000);
    let y = timeout('hello timeout 2', 5000);

    return [await x, await y]; // <--------- 修改await位置
  }

  let pre = new Date().getTime();

  foo().then(v => {
    console.log(v);

    let end = new Date().getTime();
    console.log('time: ', end - pre);
  });

  // 输出:
  // [ 'hello timeout 1', 'hello timeout 2' ]
  // time: 10029
  ```

  - 有的时候，可以将操作放到`Promise.all`中。

  ```javascript
  async function foo() {
    return await Promise.all([
      (x = timeout('timeout 1', 5000)),
      timeout('timeout 2', 5000),
    ]);
  }
  ```

# 16 Class

- Class 简介

  - 生成对象的传统方法就是通过构造函数。

    ```javascript
    function Point(x, y) {
      this.x = x;
      this.y = y;
    }

    Point.prototype.hi = function() {
      return `(${this.x}, ${this.y})`;
    };

    var p = new Point(1, 2);
    console.log(p.hi());
    ```

    - 不仅定义了构造函数，还有一个普通的成员函数。

  - 使用`ES6`的`class`关键字

    ```javascript
    class Point {
      constructor(x, y) {
        this.x = x;
        this.y = y;
      }

      hi() {
        return `(${this.x}, ${this.y})`;
      }
    }

    var p = new Point(1, 2);
    console.log(p.hi());

    console.log(typeof Point); // function
    console.log(Point === Point.prototype.constructor); // true
    ```

    - 更像是一个类，对于 C++,Java 程序员来说更好理解。
    - `ES6`的类，完全可以看作是构造函数的另一种写法。类的数据类型就是函数，类本身就指向构造函数。

  - `ES6`类内部所定义的方法，都是不可以枚举的。

    ```javascript
    class Point1 {
      constructor(x, y) {
        this.x = x;
        this.y = y;
      }

      hi() {
        return `(${this.x}, ${this.y})`;
      }
    }

    function Point2(x, y) {
      this.x = x;
      this.y = y;
    }

    Point2.prototype.hi = function() {
      return `(${this.x}, ${this.y})`;
    };

    console.log(Object.keys(Point1.prototype)); // [], 不可枚举
    console.log(Object.keys(Point2.prototype)); // ['hi']
    ```

- 类和模块内部，默认是严格模式的。
- 与`ES5`一样，实例的属性除非是显示定义在其本身(也就是定义在`this`上)，否则都是定义在原型上(也就是定义在`class`上)。

  ```javascript
  class Point {
    constructor(x, y) {
      this.x = x;
      this.y = y;
    }

    hi() {
      return `(${this.x}, ${this.y}, ${z})`;
    }
  }

  var p = new Point(1, 2);
  console.log(p.hasOwnProperty('x')); //true
  console.log(p.hasOwnProperty('y')); //true
  console.log(p.hasOwnProperty('hi')); //false
  console.log(p.__proto__.hasOwnProperty('hi')); //true
  ```

  - `x`和`y`都是实例`point`自身的属性，因为定义在`this`上，而`hi`是定义在`Point`类上。

- 和`ES5`一样，类的所有实例共享一个原型对象

  ```javascript
  var p1 = new Point(1, 2);
  var p2 = new Point(3, 4);

  console.log(p1.__proto__ === p2.__proto__); // true
  ```

- `__proto__`

  - `__proto__`并不是语言本身的特性，而是各大厂商具体实现时添加的私有属性，虽然很多现代的浏览器都支持这个私有属性，但仍然不建议在生产中使用，避免对环境产生依赖。在生产环境中，可以使用`Object.getPrototypeOf`来获取实例对象的原型。

  ```javascript
  var p1 = new Point(1, 2);
  var p2 = new Point(3, 4);

  console.log(p1.__proto__ === Object.getPrototypeOf(p2)); // true
  ```

- `Class`表达式

  - 与函数一样，类也可以使用表达式来定义。

  ```javascript
  const Point = class {
    constructor(x, y) {
      this.x = x;
      this.y = y;
    }

    hi() {
      return `(${this.x}, ${this.y})`;
    }
  };

  let p = new Point(1, 2);
  console.log(p.hi());
  ```

- `new.target属性`

  - 该属性一般用于构造函数当中，返回`new`命令作用于的那个构造函数。
  - 如果构造函数不是通过`new`调用的，那么`new.target`返回`undefined`。
  - 示例: 类构造函数必须用`new`命令调用

    ```javascript
    class Point {
      constructor() {
        if (new.target === 'undefined') {
          throw new Error('new error');
        }
      }
    }
    ```

  - 示例: 子类继承父类时，父类构造函数中的`new.target`返回的是子类

    ```javascript
    class Point {
      constructor() {
        console.log('new.target', new.target);
        console.log(new.target === Point);

        if (new.target === 'undefined') {
          throw new Error('new error');
        }
      }
    }

    class SubPoint extends Point {}

    var p = new Point(); // true
    var sp = new SubPoint(); // false
    ```

- 继承

  - `Class`可以通过`extends`关键字实现继承。
  - 子类必须在`constructor`方法中调用`super`方法，否则新建实例时报错。这是因为子类没有自己的`this`对象，而是继承父类的`this`对象。如果不调用`super`方法，子类就得不到`this`对象。且只有先调用`super`后，子类才能使用`this`。
  - `Object.getPrototypeOf`可以从子类上获取父类。

    ```javascript
    console.log(Object.getPrototypeOf(SubPoint) === Point); // true
    console.log(SubPoint.__proto__ === Point); // true
    console.log(SubPoint.prototype.__proto__ === Point.prototype); // true
    ```

    - 子类的`__proto__`属性，表示构造函数的继承，指向父类。
    - 子类的`prototype`属性的`__proto__`属性，表示方法的继承，指向父类的`prototype`。

- 实例的`__proto__`属性

  - 子类实例的`__proto__`属性的`__proto__`属性，指向父类实例的`__proto__`属性。

  ```javascript
  var p1 = new Point(1, 2);
  var p2 = new SubPoint(2, 3);

  console.log(p2.__proto__ === p1.__proto__); // false
  console.log(p2.__proto__.__proto__ === p1.__proto__); // true
  ```

- `Mixin`模式实现

  - `Mixin`模式是指，将多个类的接口混入另一个类。可以将多个对象合成一个类。

  ```javascript
  class Point {
    hi() {
      console.log('hi Point');
    }
  }

  class Color {
    bye() {
      console.log('bye Color');
    }
  }

  var copyProperties = function(target, source) {
    for (let key of Reflect.ownKeys(source)) {
      if (key !== 'constructor' && key !== 'prototype' && key !== 'name') {
        let value = Object.getOwnPropertyDescriptor(source, key);
        Object.defineProperty(target, key, value);
      }
    }
  };

  var mix = function(...mixins) {
    class Mix {}

    for (let mixin of mixins) {
      copyProperties(Mix, mixin);
      copyProperties(Mix.prototype, mixin.prototype);
    }

    return Mix;
  };

  class Item extends mix(Point, Color) {}

  var item = new Item();
  item.hi();
  item.bye();
  ```

# 17 Module

- ES6 的模块自动采用严格模式，无论有没有在模块顶部添加`'use strict';`。
  - 严格模式主要有以下限制:
    1. 变量必须声明后再使用。
    2. 函数的参数不能有同名属性。
    3. 不能使用`with`语句。
    4. 不能对只读属性复制。
    5. 不能使用前缀 0 表示八进制数。
    6. 不能用`delete prop`删除变量，只能删除属性`delete global[prop]`。
    7. `eval`不会在它的外层作用域引入变量。
    8. `eval`和`arguments`不能被重新赋值。
    9. `arguments`不会自动反映函数参数的变化。
    10. 不能使用`arguments.callee`。
    11. 不能使用`arguments.caller`。
    12. 禁止`this`指向全局对象。
    13. 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈。
    14. 增加了保留字（比如`protected`、`static`和`interface`）。
- `export`

  - 用于规定模块的对外接口。
  - 一个模块就是一个独立的文件，该文件内的所有变量，外部无法获取。
  - 如果希望外部能够获取内部的某个变量，就必须使用`export`关键字输出该变量。
  - 比如有一个文件为`config.js`，专门存储一些配置:

    ```javascript
    var config = {
      dialect: 'mysql',
      database: 'nodejs_learn',
      username: 'root',
      password: '123456',
      host: 'localhost',
      port: 3306,
    };

    export { config };
    ```

    - ES6 视`config.js`为一个模块，里面用`export`对外部输出了这个变量。

- `import`

  - 使用`export`命令定义了模块的对外接口后，其它 JS 文件就可以通过`import`命令加载这个模块。

    ```javascript
    // db.js
    import { config } from './config.js';
    ```

  - `import`还可以指定导入变量名，当然这个变量名要在`export`中找的到。还可以为导入的变量取别名。

    ```javascript
    import defaultMember from "module-name";
    import * as name from "module-name";
    import { member } from "module-name";
    import { member as alias } from "module-name";
    import { member1 , member2 } from "module-name";
    import { member1 , member2 as alias2 , [...] } from "module-name";
    import defaultMember, { member [ , [...] ] } from "module-name";
    import defaultMember, * as name from "module-name";
    import "module-name";
    ```

  - `import`命令具有提升效果，会提升到整个模块头部，首先执行。
  - `import`是静态执行的，所以不能使用表达式和变量，这些只有在运行的时候才能得到结果的语法结构。

# 18 参考资料

- [MDN JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
- [阮一峰 ECMAScript 6 入门](http://es6.ruanyifeng.com)
- [Javascript 中的 this,call,apply,bind 函数](http://alex-my.xyz/js-thi-call-apply-bind)
