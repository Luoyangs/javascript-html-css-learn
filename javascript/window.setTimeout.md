# setTimeout的一些事儿

## setTimeout()的参数：
大家都知道setInterval()和setTimeout()可以接收两个参数，第一个参数是需要回调的函数，必须传入的参数，第二个参数是时间间隔，毫秒数，可以省略。但其实他可以接收更多的参数，那么这些参数是干什么用的呢？从第三个参数开始，依次用来表示传入回调函数的参数。

```js
setTimeout(function(a,b){
   console.log(0+a+b);//这里打印的是：7
},1000,3,4);
```

> 注意：IE 9.0及以下版本，只允许setTimeout有两个参数，不支持更多的参数

**如果想向回调函数传参，可以用bind()。**

```js
setTimeout( function(a,b){}.bind(3,4), 1000 );
```

## clearTimeout
setTimeout函数，返回一个表示计数器编号的整数值，将该整数传入clearTimeout函数，就可以取消对应的定时器。
clearTimout()有以下语法: clearTimeout(timeoutID)
要使用 clearTimeout( ), 我们设定 setTimeout( ) 时, 要给予这 setTimout( ) 一个名称, 这名称就是 timeoutID , 我们叫停时, 就是用这 timeoutID来叫停, 这是一个自定义名称。

```js
var id1 = setTimeout(f,1000);  //id1就是timeoutID
var id2 = setInterval(f,1000); //id2就是timeoutID

clearTimeout(id1);
clearInterval(id2);
```

## setTimeout()的this指向：
对于javascript中的this指向问题，之前也是困扰了我好久，哎呀，哪儿有那么难嘛，其实一句话就是说:谁调用的就是指向谁啊！意思就是说调用的对象是谁this就是指向谁。
```js
var x = 1;
var obj = {
  x: 2,
  y: function(){
    console.log(this.x);
  }
};
setTimeout(obj.y,1000);  // 1
```
why?不是说了哪个对象调用的就是指向哪个对象的嘛，这里不是setTimeout函数调用了obj对象里面的y方法吗，那不还是被setTimeout调用了吗，对啊，没错啊，就是setTimeout调用的，**但是setTimeout函数是属于window的**，知道吧，所以setTimeout的对象是window，所以一切都明了了。
```js
var x = 1;
var obj = {
  x: 2,
  y: function(){
    console.log(this.x);
  }
};
setTimeout(obj.y.bind(obj),1000);  // 2
```

```js
function Animal(login) {
  this.login = login;
  this.sayHi = function() {
    console.log(this.login);  //undefined
  }
}
var dog = new Animal('John');
setTimeout(dog.sayHi, 1000);
```
等到dog.sayHi执行时，它是在全局对象中执行，但是this.login取不到值。

## setTimeout()之延迟时间为0
```js
console.log('a');
setTimeout(function(){
  console.log('b');
},0);
console.log('c');
console.log('d');
// a 
// c 
// d 
// b
```
我也不截图了。
知道为什么吗，理论上他延迟时间为0不是应该马上执行吗，不是的。**因为setTimeout运行机制说过，必须要等到当前脚本的同步任务和“任务队列”中已有的事件，全部处理完以后，才会执行setTimeout指定的任务。** 也就是说，setTimeout的真正作用是，在“任务队列”的现有事件的后面再添加一个事件，规定在指定时间执行某段代码。setTimeout添加的事件，会在下一次Event Loop执行。好吧，对事件循环不清楚的推荐看看[JavaScript 运行机制详解：再谈Event Loop](https://github.com/Luoyangs/javascript-html-css-learn/blob/master/javascript/JavaScript%20%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6%E8%AF%A6%E8%A7%A3%EF%BC%9A%E5%86%8D%E8%B0%88Event%20Loop.md)

## 事件循环中的setTimeout()
众所周知，Javascript引擎（以下简称JS引擎）是单线程的，在某一个特定的时间内只能执行一个任务，并阻塞其他任务的执行，也就是说这些任务是串行的。这样的话，用户不得不等待一个耗时的操作完成之后才能进行后面的操作，这显然是不能容忍的，但是实际开发中我们却可以使用异步代码来解决。

当异步方法比如这里的setTimeout()，或者ajax请求、DOM事件执行的时候，会交由浏览器内核的其他模块去管理。当异步的方法满足触发条件后，该模块就会将方法推入到一个任务队列中，当主线程代码执行完毕处于空闲状态的时候，就会去检查任务队列，将队列中第一个任务入栈执行，完毕后继续检查任务队列，如此循环。前提条件是主线程处于空闲状态，这就是事件循环的模型。

```js
setTimeout(function () {
    console.log("b");
},0)
console.log("a");
// a
// b
```
原理，就是上面两段话当中解释的，执行时把setTimeout()放入任务队列中去，主线程执行完主线程的任务之后去任务队列里面执行setTimeout出来执行。

```js
setTimeout(function(){
  console.log(1111);
},0)
while (true) {};
```
这里控制台是永远不会输出东西的，因为主线程已经造成了死循环，主线程一直是不会空闲的，他不会到任务队列里面去执行拿setTimeout函数来执行。

首先我们还是来看那道大家再熟悉不过的前端面试题：
```js
for (var i = 1;i <= 5;i ++) {
  setTimeout(function timer() {
    console.log(i)
  },i * 1000)
}
```
我想刚入门的童鞋或者对JS作用域、闭包以及事件循环等概念不了解的童鞋会想当然的认为这道题的答案应该是：

第一次循环，隔一秒输出1；

第二次循环，隔两秒输出2；

第三次循环，隔三秒输出3；

第四次循环，隔四秒输出4；

第五次循环，隔五秒输出5；

或者还有同学预期的结果是分别输出数字1~5，每秒一次，每次一个。

**但实际结果大家去控制台打印了都知道：以一秒的频率连续输出五个6！**

相信对于很多童鞋第一次看到这个结果是懵的，包括我第一次看到结果是懵逼的！

然而还没等你反应过来，面试官又要求你改动一下代码，要它以一秒的频率分别输出1，2，3，4，5。如果你不了解或者没有深入理解JS中的作用域、闭包以及事件循环，那么就可以和面试官说拜拜了。


这道题涉及到的知识点我上面已经提到过两次，这里我们还是先简单地过一下这些知识点：

1、作用域：这里我引用《你不知道的javascript》中的一个比喻，可以把作用域链想象成一座高楼，第一层代表当前执行作用域，楼的顶层代表全局作用域。我们在查找变量时会先在当前楼层进行查找，如果没有找到，就会坐电梯前往上一层楼，如果还是没有找到就继续向上找，以此类推。到达顶层后（全局作用域），可能找到了你所需的变量，也可能没找到，但无论如何查找过程都将停止。

2、闭包：我的理解是在传递函数类型的变量时，该函数会保留定义它的所在函数的作用域。读起来可能比较绕，或者可以简单的这么理解，A函数中定义了B函数并且它返回了B函数，那么不管B函数在哪里被调用或如何被调用，它都会保留A函数的作用域。

3、事件循环：这个概念深入起来很复杂，下面新开一个段落只说一些跟本文相关的内容。

说起事件循环，不得不提起任务队列。事件循环只有一个，但任务队列可能有多个，任务队列可分为宏任务（macro-task）和微任务（micro-task）。XHR回调、事件回调（鼠标键盘事件）、setImmediate、setTimeout、setInterval、indexedDB数据库操作等I/O以及UI rendering都属于宏任务（也有文章说UI render不属于宏任务，目前还没有定论），process.nextTick、Promise.then、Object.observer(已经被废弃)、MutationObserver(html5新特性)属于微任务。注意进入到任务队列的是具体的执行任务的函数。比如上述例子setTimeout()中的timer函数。另外不同类型的任务会分别进入到他们所属类型的任务队列，比如所有setTimeout()的回调都会进入到setTimeout任务队列，所有then()回调都会进入到then队列。当前的整体代码我们可以认为是宏任务。事件循环从当前整体代码开始第一次事件循环，然后再执行队列中所有的微任务，当微任务执行完毕之后，事件循环再找到其中一个宏任务队列并执行其中的所有任务，然后再找到一个微任务队列并执行里面的所有任务，就这样一直循环下去。这就是我所理解的事件循环。来，还是看个栗子：

```js
console.log('global')

setTimeout(function () {
   console.log('timeout1')
   new Promise(function (resolve) {
     console.log('timeout1_promise')
       resolve()
   }).then(function () {
     console.log('timeout1_then')
  })
},2000)

for (var i = 1;i <= 5;i ++) {
  setTimeout(function() {
    console.log(i)
  },i*1000)
  console.log(i)
}

new Promise(function (resolve) {
  console.log('promise1')
  resolve()
 }).then(function () {
  console.log('then1')
})

setTimeout(function () {
  console.log('timeout2')
  new Promise(function (resolve) {
    console.log('timeout2_promise')
    resolve()
  }).then(function () {
    console.log('timeout2_then')
  })
}, 1000)

new Promise(function (resolve) {
  console.log('promise2')
  resolve()
}).then(function () {
  console.log('then2')
})
```

我们来一步一步分析以上代码：

1）、首先执行整体代码，“global”会被第一个打印出来。这是**第一个输出**.

2）、执行到第一个setTimeout时，发现它是宏任务，此时会新建一个setTimeout类型的宏任务队列并派发当前这个setTimeout的回调函数到刚建好的这个宏任务队列中去，并且轮到它执行时要延迟2秒后再执行

3）、代码继续执行走到for循环，发现是循环5次setTimeout()，那就把这5个setTimeout中的回调函数依次派发到上面新建的setTimeout类型的宏任务队列中去，注意，这5个setTimeout的延迟**分别是1到5秒**。此时这个setTimeout类型的宏任务队列中应该有6个任务了。再执行for循环里的console.log(i)，很简单，直接输出1,2,3,4,5，这是**第二个输出**。

4）、再执行到new Promise，Promise构造函数中的第一个参数在new的时候会直接执行，因此不会进入任何队列，所以**第三个输出**是"promise1"，上面有说到Promise.then是微任务，那么这里会生成一个Promise.then类型的微任务队列，这里的then回调会被push进这个队列中。

5）、再继续走，执行到第二个setTimeout，发现是宏任务，派发它的回调到上面setTimeout类型的宏任务队列中去。

6）、再走到最后一个new Promise，很明显，这里会有**第四个输出**："promise2"，然后它的then中的回调也会被派发到上面的Promise.then类型的微任务队列中去。

7）、第一轮事件循环的宏任务执行完成（整体代码可以看做宏任务）。此时微任务队列中只有一个Promise.then类型微任务队列，它里面有两个任务。宏任务队列中也只有一个setTimeout类型的宏任务队列。

8）、下面执行第一轮事件循环的微任务，很明显，会分别打印出"then1"，和"then2"。分别是**第五和第六个输出**。此时第一轮事件循环完成。

9）、开始第二轮事件循环：执行setTimeout类型队列（宏任务队列）中的所有任务。发现都有延时，但延时最短的是for循环中第一次循环push进来的那个setTimeout和上面第5个步骤中的第二个setTimeout，它们都只延时1s。它们会被同时执行，但前者先被push进来，所以先执行它！它的作用就是打印变量i，在当前作用域找变量i，木有！去它上层作用域（这里是全局作用域）找，找到了，但此时的i早已是6了。（为啥不是5，那你得去补补for循环的执行流程了~）所以这里**第七个输出**是延时1s后打印出6。

10）、紧接着执行第二个setTimeout，它会先后打印出"timeout2"和"timeout2_promise"，这分别是**第八和第九个输出**。但这里发现了then，又把它push到上面已经被执行完的then队列中去。

11）、这里要注意，因为出现了微任务then队列，所以这里会执行该队列中的所有任务（此时只有一个任务），即打印出"timeout2_then"。**这是第十个输出**

11）、继续回过头来执行宏任务队列，此时是执行延时为2s的第一个setTimeout和for循环中第二次循环的那个setTimeout，跟上面一样，前者是第一个被push进来的，所以它先执行。**这里会延时1秒**（原因下面会解释）分别输出“timeout1”和“timeout1_promise”，但发现了里面也有一个then，于是push到then微任务队列并立即执行，输出了"timeout1_then"。紧接着执行for中第二次循环的setTimeout，输出6。**注意这三个几乎是同时被打印出来的。他们分别是第十一到十三个输出**。

12）、再就很简单了，把省下的for循环中后面三次循环被push进来的setTimeout依次执行，于是每隔1s输出一个6，连续输出3次。

13）、第二轮事件循环结束，全部代码执行完毕。

```
global
1
2
3
4
5
promise1
promise2
then1
then2
//延迟1s
6
timeout2
timeout2_promise
timeout2_then
//延迟1s
timeout1
17 timeout1_promise
20 timeout1_then
6
//每隔1s输出3个6
```

这里解释下为什么上面第11步不是延迟2秒再输出“timeout1”和“timeout1_promise”，这时需要理解setTimeout()延时参数的意思，**这个延迟时间始终是相对主程序执行完毕的那个时间算的 ,并且多个setTimeout执行的先后顺序也是由这个延迟时间决定的。**

再回过头来看上面那个问题，理解了事件循环的机制，问题就很简单了。for循环时setTimeout()不是立即执行的，它们的回调被push到了宏任务队列当中，而在执行任务队列里的回调函数时，变量i早已变成了6。那如何得到想要的结果呢？很简单，原理就是需要给循环中的setTimeout()创建一个闭包作用域，让它执行的时候找到的变量i是正确的。

知道了原理，解决方案就很多了，下面给出5种方案，

1）引入IIFE
```js
for(var i = 0;i<5;i ++) {
  (function(i){
    setTimeout(function timer() {
      console.log(i)
    }, i * 1000);
  })(i);
}
```

2）利用ES 6引入的let关键字
```js
for(let i = 0;i<5;i++) {
  setTimeout(function timer(){
    console.log(i);
  }, i * 1000);
}
```
for 循环头部的let 声明还会有一个特殊的行为。这个行为指出变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

3）利用ES 5引入的bind函数
```js
for (var i=1; i<=5; i++) {
  setTimeout( function timer(i) {
    console.log(i);
  }.bind(null,i), i*1000 );
}
```

4）利用setTimeout第三个参数
```js
for (var i=1; i<=5; i++) {
  setTimeout( function timer(i) {
    console.log(i);    
   }, i*1000,i );
}
```
> 注：setTimeout函数第三个参数及以后的参数都可以作为timer函数的参数。

5）把setTimeout用一个方法单独出来形成闭包
```js
var loop = function (i) {
  setTimeout(function timer() {
    console.log(i);  
  }, i*1000);
};
for (var i = 1;i <= 5; i++) {
  loop(i);
}
```
