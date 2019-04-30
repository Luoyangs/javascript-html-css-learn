# 模拟实现JS的bind方法

先看一下bind是什么?
```js
var obj = {};
obj;
typeof Function.prototype.bind; // function
typeof Function.prototype.bind();  // function
Function.prototype.bind.name;  // bind
Function.prototype.bind().name;  // bound 
console.dir(Function.prototype.bind()); // 可以自行在浏览器控制台打印看看
// f bound ()
//    length: 0
//    name: "bound "
```

## 因此可以得出结论1：
1、bind是Functoin原型链中Function.prototype的一个属性，每个函数都可以调用它。<br/>
2、bind本身是一个函数名为bind的函数，返回值也是函数，函数名是bound 。（打出来就是bound加上一个空格）。
知道了bind是函数，就可以传参，而且返回值'bound '也是函数，也可以传参，就很容易写出例子2：

后文统一 bound 指原函数original bind之后返回的函数，便于说明。
```js
var obj = {
  name: 'test',
};
function original(a, b){
  console.log(this.name);
  console.log([a, b]);
  return false;
}
var bound = original.bind(obj, 1);
var boundResult = bound(2); // 'test', [1, 2]
boundResult; // false
original.bind.name; // 'bind'
original.bind.length; // 1
original.bind().length; // 2 返回original函数的形参个数
bound.name; // 'bound original'
(function(){}).bind().name; // 'bound '
(function(){}).bind().length; // 0
```
由此可以得出结论2：
1、调用bind的函数中的this指向bind()函数的第一个参数。

2、传给bind()的其他参数接收处理了，bind()之后返回的函数的参数也接收处理了，也就是说合并处理了。

3、并且bind()后的name为bound + 空格 + 调用bind的函数名。如果是匿名函数则是bound + 空格。

4、bind后的返回值函数，执行后返回值是原函数（original）的返回值。

5、bind函数形参（即函数的length）是1。bind后返回的bound函数形参不定，根据绑定的函数原函数（original）形参个数确定。

根据结论2：我们就可以简单模拟实现一个简版bindFn
```
// 第一版 修改this指向，合并参数
Function.prototype.bindFn = function bind(thisArg){
  if(typeof this !== 'function'){
    throw new TypeError(this + 'must be a function');
  }
  // 存储函数本身
  var self = this;
  // 去除thisArg的其他参数 转成数组
  var args = [].slice.call(arguments, 1);
  var bound = function(){
    // bind返回的函数 的参数转成数组
    var boundArgs = [].slice.call(arguments);
    // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
    return self.apply(thisArg, args.concat(boundArgs));
  }
  return bound;
}
// 测试
var obj = {
  name: 'test',
};
function original(a, b){
  console.log(this.name);
  console.log([a, b]);
}
var bound = original.bindFn(obj, 1);
bound(2); // 'test', [1, 2]
bound.name; // 'bound' ?这一点怎么实现呢
```

我们知道函数是可以用new来实例化的。那么bind()返回值函数会是什么表现呢
```js
var obj = {
  name: '轩辕Rowboat',
};
function original(a, b){
  console.log('this', this); // original {}
  console.log('typeof this', typeof this); // object
  this.name = b;
  console.log('name', this.name); // 2
  console.log('this', this);  // original {name: 2}
  console.log([a, b]); // 1, 2
}
var bound = original.bind(obj, 1);
var newBoundResult = new bound(2);
console.log(newBoundResult, 'newBoundResult'); // original {name: 2} this指向了new bound()生成的新对象。

```

## 可以分析得出结论3：
1、bind原先指向obj的失效了，其他参数有效。

2、new bound的返回值是以original原函数构造器生成的新对象。original原函数的this指向的就是这个新对象,new做了什么?
```
1.创建了一个全新的对象。<br/>
2.这个对象会被执行[[Prototype]]（也就是__proto__）链接。<br/>
3.生成的新对象会绑定到函数调用的this。<br/>
4.通过new创建的每个对象将最终被[[Prototype]]链接到这个函数的prototype对象上。<br/>
5.如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用会自动返回这个新的对象。
```
所以相当于new调用时，bind的返回值函数bound内部要模拟实现new实现的操作。
```js
// 第三版 实现new调用
Function.prototype.bindFn = function bind(thisArg){
  if(typeof this !== 'function'){
    throw new TypeError(this + ' must be a function');
  }
  // 存储调用bind的函数本身
  var self = this;
  // 去除thisArg的其他参数 转成数组
  var args = [].slice.call(arguments, 1);
  var bound = function(){
    // bind返回的函数 的参数转成数组
    var boundArgs = [].slice.call(arguments);
    var finalArgs = args.concat(boundArgs);
    // new 调用时，其实this instanceof bound判断也不是很准确。es6 new.target就是解决这一问题的。
    if(this instanceof bound){
      // 这里是实现上文描述的 new 的第 1, 2, 4 步
      // 1.创建一个全新的对象
      // 2.并且执行[[Prototype]]链接
      // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
      // self可能是ES6的箭头函数，没有prototype，所以就没必要再指向做prototype操作。
      if(self.prototype){
        // ES5 提供的方案 Object.create()
        // bound.prototype = Object.create(self.prototype);
        // 但 既然是模拟ES5的bind，那浏览器也基本没有实现Object.create()
        // 所以采用 MDN ployfill方案 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create
        function Empty(){}
        Empty.prototype = self.prototype;
        bound.prototype = new Empty();
      }
      // 这里是实现上文描述的 new 的第 3 步
      // 3.生成的新对象会绑定到函数调用的`this`。
      var result = self.apply(this, finalArgs);
      // 这里是实现上文描述的 new 的第 5 步
      // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，
      // 那么`new`表达式中的函数调用会自动返回这个新的对象。
      var isObject = typeof result === 'object' && result !== null;
      var isFunction = typeof result === 'function';
      if(isObject || isFunction){
        return result;
      }
      return this;
    }
    else{
      // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
      return self.apply(thisArg, finalArgs);
    }
  };
  return bound;
}
```

注释中提到this instanceof bound也不是很准确，ES6 new.target很好的解决这一问题
```js
function Student(name){
  if(this instanceof Student){
    this.name = name;
    console.log('name', name);
  }
  else{
    throw new Error('必须通过new关键字来调用Student。');
  }
}
var student = new Student('轩辕');
var notAStudent = Student.call(student, 'Rowboat'); // 不抛出错误，且执行了。
console.log(student, 'student', notAStudent, 'notAStudent');

function Student2(name){
  if(typeof new.target !== 'undefined'){
    this.name = name;
    console.log('name', name);
  }
  else{
    throw new Error('必须通过new关键字来调用Student2。');
  }
}
var student2 = new Student2('轩辕');
var notAStudent2 = Student2.call(student2, 'Rowboat');
console.log(student2, 'student2', notAStudent2, 'notAStudent2'); // 抛出错误
```

细心的同学可能会发现了这版本的代码没有实现bind后的bound函数的nameMDN Function.name和lengthMDN Function.length
```js
Object.defineProperties(bound, {
  'length': {
    value: self.length,
  },
  'name': {
    value: 'bound ' + self.name,
  }
});
```

最后es5-shim的源码实现bind，直接附上源码（有删减注释和部分修改等）
```js
var $Array = Array;
var ArrayPrototype = $Array.prototype;
var $Object = Object;
var array_push = ArrayPrototype.push;
var array_slice = ArrayPrototype.slice;
var array_join = ArrayPrototype.join;
var array_concat = ArrayPrototype.concat;
var $Function = Function;
var FunctionPrototype = $Function.prototype;
var apply = FunctionPrototype.apply;
var max = Math.max;
// 简版 源码更复杂些。
var isCallable = function isCallable(value){
    if(Object.prototype.toString.call(value) !== '[Object Function]'){
      return false;
    }
    return true;
};
var Empty = function Empty() {};
// 源码是 defineProperties
// 源码是bind笔者改成bindFn便于测试
FunctionPrototype.bindFn = function bind(that) {
  var target = this;
  if (!isCallable(target)) {
      throw new TypeError('Function.prototype.bind called on incompatible ' + target);
  }
  var args = array_slice.call(arguments, 1);
  var bound;
  var binder = function () {
    if (this instanceof bound) {
      var result = apply.call(
        target,
        this,
        array_concat.call(args, array_slice.call(arguments))
      );
      if ($Object(result) === result) {
        return result;
      }
      return this;
    } else {
      return apply.call(
        target,
        that,
        array_concat.call(args, array_slice.call(arguments))
      );
    }
  };
  var boundLength = max(0, target.length - args.length);
  var boundArgs = [];
  for (var i = 0; i < boundLength; i++) {
    array_push.call(boundArgs, '$' + i);
  }
  // 这里是Function构造方式生成形参length $1, $2, $3...
  bound = $Function('binder', 'return function (' + array_join.call(boundArgs, ',') + '){ return binder.apply(this, arguments); }')(binder);

  if (target.prototype) {
    Empty.prototype = target.prototype;
    bound.prototype = new Empty();
    Empty.prototype = null;
  }
  return bound;
};
```

[原文](https://segmentfault.com/a/1190000017091983)