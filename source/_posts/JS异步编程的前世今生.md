---
title: JS异步编程的前世今生
subtitle: "由于JavaScript是单线程的语言，因此异步编程对于js的重要程度可想而知，可以说没有异步的js程序寸步难行。"
cover: https://raw.githubusercontent.com/leaf930814/blog-misc/master/img/asynchronous_programming.png
date: 2018-05-17 19:25:02
categories: Web开发
tags:
  - 异步编程
author:
    nick: leaf++
    github_name: leaf930814
---

由于JavaScript是单线程的语言，因此异步编程对于js的重要程度可想而知，可以说没有异步的js程序寸步难行。本文是我在学习阮一峰大神的《深入掌握 ECMAScript 6 异步编程》以及《ES6标准入门》结合实际工作的收获，分享给广大网友共同学习。
什么是异步在这里就不赘述了，还不了解的小伙伴建议先去看看异步的概念。本文将以时间轴的顺序来讲述异步调用方案的演变，和我的一些感受。文中的一些代码以及部分概念会直接引用文章里的，毕竟在大神面前没有必要再班门弄斧了。
#### 最开始的异步实现方案--回调函数（callback）
最早的异步处理方案是回调函数，一段异步程序执行完成后，执行回调函数里的语句。如读取文件的处理:
```js
fs.readFile('/etc/passwd', function (err, data) {
  if (err) throw err;
  console.log(data);
})
```
回调函数最大的问题就是回调函数噩梦也叫回调地狱（callback hell）。接下来的一段代码，能让你深刻的体会到什么叫做回调地狱。
```js
fs.readFile(fileA, function (err, data) {
  if(err) throw err
  fs.readFile(fileB, function (err, data) {
    if(err) throw err
    fs.readFile(fileC,function(err,data){
      fs.readFile(fileD,function(err,data){
        fs.readFile(fileE,function(err,data){
          console.log(data)
        })
      })      
    })  
  })
})
```
为了形象的体现出回调函数方法的弊端，这里特意用了好几层异步操作。要知道，在实际的程序里，异步的连续操作是很常见的现象。可以看到上面的代码有以下缺陷：
1、可读性和维护性惨不忍睹
2、很容易造成变量污染
但在我看来，这种写法也不是一无是处，至少它的语义化还是比较强的，让人容易理解和使用。
为了解决回调地狱这个问题，Promise对象应运而生。
#### 更好异步写法--promise
有了Promise，上面的代码就可以这样写：
```js
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(data=>{
  console.log(data.toString());
})
.catch(err=>{
  console.log(err)
})
.then(()=>{
  return readFile(fileB);
})
.catch(err=>{
  console.log()
})
.then(data=>{
  console.log(data.toString());
})
...
```
这样写，很明显，代码的可读性提高了不止一个档次，各个异步操作一目了然。在引入了ES6的新语法后，更是简洁明了。
Promise确实很好的解决了回调地狱的问题，目前也已经很广泛的在使用，像我的工作中大部分场合的异步就是用promise。
但是在我看来它的缺点是不太好理解，也可以说成不够语义化，至少我在刚开始接触的时候，花了很长一段时间才搞清楚Promise对象和then的用法。
另外一点，引用阮老师的说法，Promise写法只能算是回调函数的改进，只是提高了代码的可维护性，除此之外，并无新意，也就是说，它没有从思想上去改进异步编程的实现方案。
#### 新一代异步方案--Generator函数
Genorator函数形式如下:
```js
function* gen(x){
  var y = yield x + 2;
  return y;
}
```
其中yield是一个命令，英文直译是“产出”的意思。Generator函数的调用方法和普通的一样，函数名后面接括号就可以调用。但它和普通函数的区别是，调用这个函数不会直接执行函数语句，而是返回一个指针对象。对这个指针对象使用next()方法，才能真正执行函数里的语句。
比如，如下的代码:
```js
var g = gen(1);
g.next() // { value: 3, done: false }
g.next() // { value: undefined, done: true }
```
gen(1)调用的时候，不会执行gen()函数里的任何语句，仅仅只是返回一个指针。只有当g.next()被调用的时候，才会开始执行函数体里面的语句。
*这里需要澄清一下语句和表达式的区别。在我看来，所谓程序，就是输入+输出+控制三个过程。表达式就是一个提供或者不提供输入，然后按照一定的处理逻辑得到输出的短语。如3+2，就是给出3和2两个输入值，按照相加的逻辑得到5的输出，或者直接是一个值的形式，也是一个表达式，这个表达式的输入是这个值，处理逻辑是什么都不做，所以输出也是自己。或者是一个函数调用，填入参数作为输入或者不填参数，函数体作为一个集成的处理逻辑，最后得到函数的返回值。而语句，顾名思义，是一个完整的任务执行过程，也就是控制的过程，一般里面会包含了很多的表达式，比如赋值操作，声明变量，if语句，循环语句等等。当然，以上只是鄙人的大致理解，有不妥当和不准确之处，敬请见谅。*
yield命令需要放在语句之中，独立的表达式之前。如前面的var y=yield x+2，显然x+2是一个表达式，y=x+2是一个语句。
如果把yield放在js语句之前，就会报错，如:
```js
yield y=x+2//Uncaught SyntaxError
```
如果把yield放在js表达式之中，也会报错
```js
y=x+yield 2//Uncaught SyntaxError
```
如果给表达式之中加个括号，就可以：
```js
y=x+(yield 2)//正确
```
因为加了括号后，括号里面就算是一个独立的表达式了，那么yield自然就是算放在了表达式的前面而不是中间了。

g.next()方法被调用后，函数体里的语句开始执行，从第一句开始依次往下执行，跟普通的js语句是一样的，这时，遇到了第一个有yield命令的语句。
此时，从这条包含yield的语句开始，下面的语句都会暂停执行，包括这条包含yield的语句本身，也不会执行。
此时g.next()返回的是一个对象，对象里包含了两个属性。value属性返回的是yield命令紧跟的后面的表达式返回的值，done表达式表示函数体里面的语句是否已经执行完。
这里需要注意的是，对象里的value虽然是yield命令后面表达式的返回值，但这并不是说包含yield的那条语句就执行了，相反，那条语句并没有执行，或者可以简单地理解为，仅仅只有yield的后面的表达式执行了。
然后，再调用一次g.next()方法，函数体会接着包含yield的语句的下面的一条语句继续执行，直到遇到下一个下一个包含yield的语句，函数体再次暂停，如果没有再次遇到，则一直执行完，返回一个和前面一样形式的包含value和yield的对象，如果函数里有return，则value是return后面的表达式的值，如没有没有return，则value是undefined。此时属性done的值为true，表示函数体已经全部执行完。接下来，调用g.next()方法都是返回{value:undefined,done:true}对象。
再次把整个函数和整个执行过程以及结果贴一遍以方便查看，因为接下来我想讨论一个问题。
```js
function* gen(x){
  var y = yield x + 2;
  return y;
}
var g=gen(1)
g.next()//{value:3,done:false}
g.next()//{value:undefined,done:true}
```
请仔细看两次的g.next()结果，第一次的value是3，没有问题，是x+2返回的值，但是，请看第二次返回的结果，value竟然是undefined!要知道，函数明明是有返回值的，那就是y，而y是等于x+2的，也就是应该是3的，为什么是undefined呢？
**以下，是我的解释。
之所以明明有返回值y，却得到的是undefined，是因为，包含yield命令的那一句语句，会被拆分开来，yield与后面的表达式一起合成了一个语句，这个语句的执行结果就是在g.next()方法调用后返回一个对象，然后把表达式的值存进对象的value属性里。而语句执行完了之后，显然是只会返回undefined的，这也就意味着在函数体内y=yield x+2这句话，就变成了y=undefined了，看到这里，相信大家会有疑惑了，那这样的赋值还有什么意义？其实刚刚我的就是只说了一半，另一半还没有说，接下来就是另一半了。前面说第一次调用g.next()方法后，函数体内部就好比变成了y=undefined,这时是没有问题的，但是玄妙之处就在下一次的g.next()调用，先直接说现象,第二次如果我传一个参数给next呢，也就是调用g.next(5)呢，结果是返回了一个{value:5,done:true}对象，也就是说，在下一次调用g.next()的时候，如果传入参数，那么参数就会替换上一次的yield和后面表达式组成的语句执行结果的值，也就是undefined的值，所以就变成了y=5，那么自然后面返回的y的值就是5了。
换一句话来说，如果想要按照函数体的语句来正常执行的话，那么应该这么写:**
```js
function* gen(x){
  var y = yield x + 2;
  return y;
}
var g=gen(1)
var result1=g.next()
console.log(result1)//{value:3,done:false}
var result2=g.next(result1.value)
console.log(result2)//{value:3,done:true}
```
也就是说把前面一次执行的该有的结果再传进第二次的next参数里面。

这个机制虽然在一开始有点不习惯，但是它是很有好处的，那就是可以在generator函数运行的不同阶段往里面动态的注入自己想要传入的值。
**注：第一次使用next方法时，往里面传入的值是无效的。**

generator函数的优点是引入了新的协程的概念，使代码的写法几乎与同步没有什么区别，另外也使代码更易于阅读和理解，能一眼就看出哪些地方有异步的操作，以及对异步做怎样的处理。但generator的缺点也很明显，那就是很原生（自己的理解），对于函数里的执行时机需要完全自己去写逻辑控制，需要自己判断什么时候异步操作执行完成，然后还要自己去调用下一步的操作，总的来说就是不够智能。

##### co模块
co模块就是一个智能的generator自动执行器,只需要把generator函数传入co函数，函数体内的语句就能自动执行，完全不用考虑执行时机的问题。co函数返回一个Promise对象，因此还可以用then方法来添加回调函数.使用起来很简单:
```js
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
var co = require('co');
co(gen).then(function (){
  console.log('Generator 函数执行完成');
});
```
**处理并发的异步操作**
并发的异步就是说几个异步同时开始进行，而不是排队一个一个进行。写法如下：
```js
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);
}).catch(onerror);

// 对象的写法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res);
}).catch(onerror);
```

#### 终极异步方案（或许）--async函数
引用阮老师原文对async的描述：
> async 函数是什么？一句话，它就是 Generator 函数的语法糖。
> 1. 内置执行器
Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。
> 2. 更好的语义
async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。
> 3. 更广的适用性
co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。
>4. 返回值是Promise
async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。

##### async函数的使用
async函数的使用也非常简单:
```js
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

最后，贴出使用async函数的注意点:
1. 正常情况下，await命令后面是一个 Promise对象。如果不是，会被转成一个立即resolve的 Promise 对象。所以运行结果可能是rejected，最好把await命令放在try...catch代码块中。
2. 多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。有以下两种写法:
``` js
 // 写法一,比较好理解
let [foo, bar] = await Promise.all([getFoo(), getBar()]);
// 写法二，结构更加清晰
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

3. await命令只能用在async函数之中，如果用在普通函数，就会报错

关于更多关于异步编程的一些实现原理和细节的东西，阮一峰老师的es6入门教程和博客里写得非常详尽，就不再班门弄斧了。
最后，再次感谢阮老师的教程，让我受益匪浅。
