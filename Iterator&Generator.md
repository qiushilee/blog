# Iterator 迭代器 ECMAScript 2015 (ES6) 
## 为什么会被添加到 JavaScript
循环问题
```
var colors = ["red", "green", "blue"];

for (var i = 0, len = colors.length; i < len; i++) {
    console.log(colors[i]);
}
```
虽然这个循环是相当简单的，当嵌套它们并且需要跟踪多个变量时，循环的复杂性增加，额外的复杂性可能导致错误。  Iterator 意在解决这个问题。

## 概念
JavaScript原有的表示“集合”的数据结构，主要是数组（Array）和对象（Object），ES6又添加了Map和Set。这样就有了四种数据集合，用户还可以组合使用它们，定义自己的数据结构，比如数组的成员是Map，Map的成员是对象。这样就需要一种统一的接口机制，来处理所有不同的数据结构。
遍历器（Iterator）就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。

Iterator的作用有三个：
* 为各种数据结构，提供一个统一的、简便的访问接口；
* 使得数据结构的成员能够按某种次序排列；
* ES6创造了一种新的遍历命令for...of循环，Iterator接口主要供for...of消费。

下面是一个模拟 Iterator 的例子
```
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}
```
由于Iterator只是把接口规格加到数据结构之上，所以，遍历器与它所遍历的那个数据结构，实际上是分开的

# Generator
Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同。
返回Iterator。 Generator函数在`function`关键字后面用星号(`*`)表示，并使用新的`yield`关键字。 如下例所示：
```
function* idMaker(){
  var index = 0;
  while(true)
  yield index++;
}

var gen = idMaker(); // "Generator { }"

console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
// ...
```

## yield语句
yield语句就是暂停标志。
遍历器对象的next方法的运行逻辑如下：
* 遇到yield语句，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。
* 下一次调用next方法时，再继续往下执行，直到遇到下一个yield语句。
* 如果没有再遇到新的yield语句，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。
* 如果该函数没有return语句，则返回的对象的value属性值为undefined。

需要注意的是，yield语句后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为JavaScript提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。

## yield* 语句
用来在一个Generator函数里面执行另一个Generator函数。

## 异步编程
异步编程对JavaScript语言太重要。Javascript语言的执行环境是“单线程”的，如果没有异步编程，根本没法用，非卡死不可。

ES6诞生以前，异步编程的方法，大概有下面四种。
* 回调函数
* 事件监听
* 发布/订阅
* Promise 对象

ES6将JavaScript异步编程带入了一个全新的阶段，ES7的Async函数更是提出了异步编程的终极解决方案。

Generator 异步编程例子

Promise 版本
```
function request(url) {
  return new Promise(function(resolve, reject) {
    var xhr = new XMLHttpRequest();
    xhr.addEventListener('load', function() {
      try {
        resolve(JSON.parse(this.responseText));
      } catch(e) { reject(e); }
    });
    xhr.open('GET', url);
    xhr.send();
  });
}

function Users(userName = '') {
  return request(`http://localhost/users/${userName}`);
}

function Posts(userID) {
  return request(`http://localhost/posts/${userID}`);
}

function Friends(postID) {
  return request(`http://localhost/friends/${postID}`);
}

function get() {
  Users('test')
  .then(user => Posts(user.id))
  .then(post => Friends(post.id))
  .then(friend => console.log(friend));
}
get();
```

Generator 版本
```
function* get() {
  let user = yield Users('test');
  let post = yield Posts(user.id);
  let friend = yield Friends(post.id);
  console.log(friend);
}
let g = get();
g.next().value.then((user) => {
  g.next(user).value.then((post) => {
    g.next(post).value.then((friend) => {
      g.next(friend);
    });
  });
});
```

自动执行器版本
```
function run(generator) {
  const gen = generator();
  gogogo(gen.next());
  function gogogo(result) {
    if(!result.done) {
      try {
        result.value.then(resp => gogogo(gen.next(resp)));
      } catch(e) {
        gogogo(gen.next());
      }
    }
  }
}

run(get);
```

## `Generator.prototype.throw()` 错误处理
throw（）方法通过向其中抛出一个错误来恢复 generator 的执行，并返回具有两个属性done和value的对象。

## co
co 是著名程序员TJ Holowaychuk于2013年6月发布的一个小工具，用于Generator函数的自动执行。
```
const co = require('co');
co(get).then(() => console.log('get() 执行完毕'));
```

# 引用
* [Iterators and Generators](https://github.com/nzakas/understandinges6/blob/master/manuscript/08-Iterators-And-Generators.md)
* [Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)
* [Iterator和for...of循环](http://es6.ruanyifeng.com/#docs/iterator)
* [Generator函数](http://es6.ruanyifeng.com/#docs/generator)
* [Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)
* [function*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
* [yield](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield)
* [yield*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*)
* [异步操作和Async函数](http://es6.ruanyifeng.com/#docs/async)
