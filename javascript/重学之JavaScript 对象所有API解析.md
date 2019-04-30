# 重学之JavaScript 对象所有API解析

创建对象的两种方式：
```js
var o = new Object();
var o = {}; // 推荐
```

该构造器可以接受任何类型的参数，并且会自动识别参数的类型，并选择更合适的构造器来完成相关操作。比如：
```js
var o = new Object('something');
o.constructor; // ƒ String() { [native code] }
var n = new Object(123);
n.constructor; // ƒ Number() { [native code] }
n.constructor === Number; // true
```

## Object构造器的成员

### Object.prototype
该属性是所有对象的原型（包括 Object对象本身），语言中的其他对象正是通过对该属性上添加东西来实现它们之间的继承关系的。所以要小心使用。
比如
```js
var s = new String('xuanyuan');
Object.prototype.custom = 1;
console.log(s.custom); // 1
```

## Object.prototype 的成员

### Object.prototype.constructor
该属性指向用来构造该函数对象的构造器，在这里为Object()
```js
Object.prototype.constructor === Object; // true
var o = new Object();
o.constructor === Object; // true
```

### Object.prototype.toString(radix)
该方法返回的是一个用于描述目标对象的字符串。特别地，当目标是一个Number对象时，可以传递一个用于进制数的参数radix，该参数radix，该参数的默认值为10。
```js
var o = {prop:1};
o.toString(); // '[object Object]'
var n = new Number(255);
n.toString(); // '255'
n.toString(16); // 'ff'
```

### Object.prototype.toLocaleString()
该方法的作用与toString()基本相同，只不过它做一些本地化处理。该方法会根据当前对象的不同而被重写，例如Date(),Number(),Array(),它们的值都会以本地化的形式输出。当然，对于包括Object()在内的其他大多数对象来说，该方法与toString()是基本相同的。

在浏览器环境下，可以通过BOM对象Navigator的language属性（在IE中则是userLanguage）来了解当前所使用的语言：
```js
navigator.language; //'en-US'
```

### Object.prototype.valueOf()
该方法返回的是用基本类型所表示的this值，如果它可以用基本类型表示的话。如果Number对象返回的是它的基本数值，而Date对象返回的是一个时间戳（timestamp）。如果无法用基本数据类型表示，该方法会返回this本身。
```js
// Object
var o = {};
typeof o.valueOf(); // 'object'
o.valueOf() === o; // true
// Number
var n = new Number(101);
typeof n; // 'object'
typeof n.vauleOf; // 'function'
typeof n.valueOf(); // 'number'
n.valueOf() === n; // false
// Date
var d = new Date();
typeof d.valueOf(); // 'number'
d.valueOf(); // 1503146772355
```

### Object.prototype.hasOwnProperty(prop)
该方法仅在目标属性为对象自身属性时返回true,而当该属性是从原型链中继承而来或根本不存在时，返回false。

var o = {prop:1};
o.hasOwnProperty('prop'); // true
o.hasOwnProperty('toString'); // false
o.hasOwnProperty('formString'); // false


### Object.prototype.isPrototypeOf(obj)
如果目标对象是当前对象的原型，该方法就会返回true，而且，当前对象所在原型上的所有对象都能通过该测试，并不局限与它的直系关系。
```js
var s = new String('');
Object.prototype.isPrototypeOf(s); // true
String.prototype.isPrototypeOf(s); // true
Array.prototype.isPrototypeOf(s); // false
```

### Object.prototype.propertyIsEnumerable(prop)
如果目标属性能在for in循环中被显示出来，该方法就返回true
```js
var a = [1,2,3];
a.propertyIsEnumerable('length'); // false
a.propertyIsEnumerable(0); // true
```

## 在ES5中附加的Object属性
在ES3中，除了一些内置属性（如：Math.PI），对象的所有的属性在任何时候都可以被修改、插入、删除。在ES5中，我们可以设置属性是否可以被改变或是被删除——在这之前，它是内置属性的特权。ES5中引入了属性描述符的概念，我们可以通过它对所定义的属性有更大的控制权。这些属性描述符（特性）包括：
* value——当试图获取属性时所返回的值。
* writable——该属性是否可写。
* enumerable——该属性在for in循环中是否会被枚举
* configurable——该属性是否可被删除。
* set()——该属性的更新操作所调用的函数。
* get()——获取属性值时所调用的函数。

另外，数据描述符（其中属性为：enumerable，configurable，value，writable）与存取描述符（其中属性为enumerable，configurable，set()，get()）之间是有互斥关系的。在定义了set()和get()之后，描述符会认为存取操作已被 定义了，其中再定义value和writable会引起错误。
以下是ES3风格的属性定义方式：
```js
var person = {};
person.legs = 2;
```
以下是等价的ES5通过数据描述符定义属性的方式：
```js
var person = {};
Object.defineProperty(person, 'legs', {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
});
```
其中， 除了value的默认值为undefined以外，其他的默认值都为false。这就意味着，如果想要通过这一方式定义一个可写的属性，必须显示将它们设为true。
或者，我们也可以通过ES5的存储描述符来定义：
```js
var person = {};
Object.defineProperty(person, 'legs', {
    set:function(v) {
        return this.value = v;
    },
    get: function(v) {
        return this.value;
    },
    configurable: true,
    enumerable: true
});
person.legs = 2;
```
这样一来，多了许多可以用来描述属性的代码，如果想要防止别人篡改我们的属性，就必须要用到它们。此外，也不要忘了浏览器向后兼容ES3方面所做的考虑。例如，跟添加Array.prototype属性不一样，我们不能再旧版的浏览器中使用shim这一特性。

另外，我们还可以（通过定义nonmalleable属性），在具体行为中运用这些描述符：
```js
var person = {};
Object.defineProperty(person, 'heads', {value: 1});
person.heads = 0; // 0
person.heads; // 1  (改不了)
delete person.heads; // false
person.heads // 1 (删不掉)
```

### Object.defineProperty(obj, prop, descriptor) (ES5)
> Vue.js文档：如何追踪变化 把一个普通 JavaScript 对象传给 Vue 实例的 data 选项，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。Object.defineProperty 是仅 ES5 支持，且无法 shim 的特性，这也就是为什么 Vue 不支持 IE8 以及更低版本浏览器的原因。

### Object.defineProperties(obj, props) (ES5)
该方法的作用与defineProperty()基本相同，只不过它可以用来一次定义多个属性。
比如：
```js
var glass = Object.defineProperties({}, {
  'color': {
      value: 'transparent',
      writable: true
  },
  'fullness': {
      value: 'half',
      writable: false
  }
});
glass.fullness; // 'half'
```

### Object.getPrototypeOf(obj) (ES5)
之前在ES3中，我们往往需要通过Object.prototype.isPrototypeOf()去猜测某个给定的对象的原型是什么，如今在ES5中，我们可以直接询问改对象“你的原型是什么？”
```js
Object.getPrototypeOf([]) === Array.prototype; // true
Object.getPrototypeOf(Array.prototype) === Object.prototype; // true
Object.getPrototypeOf(Object.prototype) === null; // true
```

### Object.create(obj, descr) (ES5)
该方法主要用于创建一个新对象，并为其设置原型，用（上述）属性描述符来定义对象的原型属性。
```js
var o = Object.create(parent, {
    prop: {
        value: 1
    }
});
o.hi; // 'Hello'
// 获得它的原型
Object.getPrototypeOf(parent) === Object.prototype; // true 说明parent的原型是Object.prototype
Object.getPrototypeOf(o); // {hi: "Hello"} // 说明o的原型是{hi: "Hello"}
o.hasOwnProperty('hi'); // false 说明hi是原型上的
o.hasOwnProperty('prop'); // true 说明prop是原型上的自身上的属性。
```
现在，我们甚至可以用它来创建一个完全空白的对象，这样的事情在ES3中可是做不到的。
```js
var o = Object.create(null);
typeof o.toString(); // 'undefined'
```

### Object.getOwnPropertyDesciptor(obj, property) (ES5)
该方法可以让我们详细查看一个属性的定义。甚至可以通过它一窥那些内置的，之前不可见的隐藏属性。
```js
Object.getOwnPropertyDescriptor(Object.prototype, 'toString');
// {writable: true, enumerable: false, configurable: true, value: ƒ toString()}
```


### Object.getOwnPropertyNames(obj) (ES5)
该方法返回一个数组，其中包含了当前对象所有属性的名称（字符串），不论它们是否可枚举。当然，也可以用Object.keys()来单独返回可枚举的属性。
```js
Object.getOwnPropertyNames(Object.prototype);
// ["__defineGetter__", "__defineSetter__", "hasOwnProperty", "__lookupGetter__", "__lookupSetter__", "propertyIsEnumerable", "toString", "valueOf", "__proto__", "constructor", "toLocaleString", "isPrototypeOf"]
Object.keys(Object.prototype);
// []
Object.getOwnPropertyNames(Object);
// ["length", "name", "arguments", "caller", "prototype", "assign", "getOwnPropertyDescriptor", "getOwnPropertyDescriptors", "getOwnPropertyNames", "getOwnPropertySymbols", "is", "preventExtensions", "seal", "create", "defineProperties", "defineProperty", "freeze", "getPrototypeOf", "setPrototypeOf", "isExtensible", "isFrozen", "isSealed", "keys", "entries", "values"]
Object.keys(Object);
// []
```

### Object.preventExtensions(obj) (ES5) & Object.isExtensible(obj) (ES5)
preventExtensions()方法用于禁止向某一对象添加更多属性，而isExtensible()方法则用于检查某对象是否还可以被添加属性。
```js
var deadline = {};
Object.isExtensible(deadline); // true
deadline.date = 'yesterday'; // 'yesterday'
Object.preventExtensions(deadline);
Object.isExtensible(deadline); // false
deadline.date = 'today';
deadline.date; // 'today'
// 尽管向某个不可扩展的对象中添加属性不算是一个错误操作，但它没有任何作用。
deadline.report = true;
deadline.report; // undefined
```

### Object.seal(obj) (ES5) & Object.isSeal(obj) (ES5)
seal()方法可以让一个对象密封，并返回被密封后的对象。
seal()方法的作用与preventExtensions()基本相同，但除此之外，它还会将现有属性
设置成不可配置。也就是说，在这种情况下，我们只能变更现有属性的值，但不能删除或（用defineProperty()）重新配置这些属性，例如不能将一个可枚举的属性改成不可枚举。
```js
var person = {legs:2};
person === Object.seal(person); // true
Object.isSealed(person); // true
Object.getOwnPropertyDescriptor(person, 'legs');
// {value: 2, writable: true, enumerable: true, configurable: false}
delete person.legs; // false (不可删除，不可配置)
Object.defineProperty(person, 'legs',{value:2});
person.legs; // 2
person.legs = 1;
person.legs; // 1 (可写)
Object.defineProperty(person, "legs", { get: function() { return "legs"; } });
// 抛出TypeError异常
Object.defineProperty(person, "age", { get: function() { return 12; } });
// TypeError: Cannot define property age, object is not extensible
```


### Object.freeze(obj) (ES5) & Object.isFrozen(obj) (ES5)
freeze()方法用于执行一切不受seal()方法限制的属性值变更。Object.freeze() 方法可以冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。
```js
var deadline = Object.freeze({date: 'yesterday'});
deadline.date = 'tomorrow';
deadline.excuse = 'lame';
deadline.date; // 'yesterday'
deadline.excuse; // undefined
Object.isSealed(deadline); // true;
Object.isFrozen(deadline); // true
Object.getOwnPropertyDescriptor(deadline, 'date');
// {value: "yesterday", writable: false, enumerable: true, configurable: false} (不可配置，不可写)
Object.keys(deadline); // ['date'] (可枚举)
```

### Object.keys(obj) (ES5)
该方法是一种特殊的for-in循环。它只返回当前对象的属性（不像for-in），而且这些属性也必须是可枚举的（这点和Object.getOwnPropertyNames()不同，不论是否可以枚举）。返回值是一个字符串数组。
```js
Object.prototype.customProto = 101;
Object.getOwnPropertyNames(Object.prototype);
// [..., "constructor", "toLocaleString", "isPrototypeOf", "customProto"]
Object.keys(Object.prototype); // ['customProto']


var o = {own: 202};
o.customProto; // 101
Object.keys(o); // ['own']
```


## 在ES6中附加的Object属性

### Object.is(value1, value2) (ES6)
该方法用来比较两个值是否严格相等。它与严格比较运算符（===）的行为基本一致。
不同之处只有两个：一是+0不等于-0，而是NaN等于自身
```js
bject.is('xuanyuan', 'xuanyuan'); // true
Object.is({},{}); // false
Object.is(+0, -0); // false
+0 === -0; // true
Object.is(NaN, NaN); // true
NaN === NaN; // false
```
ES5可以通过以下代码部署Object.is
```js
Object.defineProperty(Object, 'is', {
  value: function() {x, y} {
    if (x === y) {
      // 针对+0不等于-0的情况
      return x !== 0 || 1 / x === 1 / y;
    }
    // 针对 NaN的情况
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true
});
```

### Object.assign(target, ...sources) (ES6)
该方法用来源对象（source）的所有可枚举的属性复制到目标对象（target）。它至少需要两个对象作为参数，第一个参数是目标对象target，后面的参数都是源对象（source）。只有一个参数不是对象，就会抛出TypeError错误。
```js
var target = {a: 1};
var source1 = {b: 2};
var source2 = {c: 3};
obj = Object.assign(target, source1, source2);
target; // {a:1,b:2,c:3}
obj; // {a:1,b:2,c:3}
target === obj; // true
// 浅拷贝
obj.a = 100;
target; // {a:100,b:2,c:3} 

// 如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。
var source3 = {a:2,b:3,c:4};
Object.assign(target, source3);
target; // {a:2,b:3,c:4}

// 方法也是可以复制的
var source4 = {a:2,b:function() {}};
Object.assign(target, source4);
target; // {a, b: function() {}}
```
> Object.assign只复制自身属性，不可枚举的属性（enumerable为false）和继承的属性不会被复制。
```js
Object.assign({b: 'c'}, 
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  });
);
// {b: 'c'}
```
属性名为Symbol值的属性，也会被Object.assign()复制。
```js
Object.assign({a: 'b'}, {[Symbol('c')]: 'd'});
// {a: 'b', Symbol(c): 'd'}
```
对于嵌套的对象，Object.assign()的处理方法是替换，而不是添加。
```js
Object.assign({a: {b:'c',d:'e'}}, {a:{b:'hello'}});
// {a: {b:'hello'}}
```
对于数组，Object.assign()把数组视为属性名为0、1、2的对象。
```js
Object.assign([1,2,3], [4,5]);
// [4,5,3]
```


### Object.getOwnPropertySymbols(obj) (ES6)
该方法会返回一个数组，该数组包含了指定对象自身的（非继承的）所有 symbol 属性键。
该方法和 Object.getOwnPropertyNames() 类似，但后者返回的结果只会包含字符串类型的属性键，也就是传统的属性名。
```js
Object.getOwnPropertySymbols({a: 'b', [Symbol('c')]: 'd'});
// [Symbol(c)]
```

### Object.setPrototypeOf(obj, prototype) (ES6)
该方法设置一个指定的对象的原型 ( 即, 内部[[Prototype]]属性）到另一个对象或 null。
__proto__属性用来读取或设置当前对象的prototype对象。目前，所有浏览器（包括IE11）都部署了这个属性。
```js
// ES3写法
var obj = {
    method: function(){
        // code ...
    }
};
// obj.__proto__ = someOtherObj;
// ES5写法
var obj = Object.create(someOtherObj);
obj.method = function(){
    // code ...
};
```
该属性没有写入ES3的正文，而是写入了附录。__proto__前后的双下划线说明它本质上是一个内部属性，而不是正式对外的一个API。无论从语义的角度，还是从兼容性的角度，都不要使用这个属性。而是使用Object.setPrototypeOf()（写操作），Object.getPrototypeOf()（读操作），或Object.create()（生成操作）代替。
在实现上，__proto__调用的Object.prototype.__proto__。
Object.setPrototypeOf()方法的作用与__proto__作用相同，用于设置一个对象的prototype对象。它是ES6正式推荐的设置原型对象的方法。


## 在ES8中附加的Object属性
### Object.getOwnPropertyDescriptors(obj) (ES8)
该方法基本与Object.getOwnPropertyDescriptor(obj, property)用法一致，只不过它可以用来获取一个对象的所有自身属性的描述符。
```
Object.getOwnPropertyDescriptor(Object.prototype, 'toString');
// {writable: true, enumerable: false, configurable: true, value: ƒ toString()}
Object.getOwnPropertyDescriptors(Object.prototype);
// constructor: {value: ƒ, writable: true, enumerable: false, configurable: true}
hasOwnProperty: {value: ƒ, writable: true, enumerable: false, configurable: true}
isPrototypeOf: {value: ƒ, writable: true, enumerable: false, configurable: true}
propertyIsEnumerable: {value: ƒ, writable: true, enumerable: false, configurable: true}
toLocaleString: {value: ƒ, writable: true, enumerable: false, configurable: true}
toString: {value: ƒ, writable: true, enumerable: false, configurable: true}
valueOf: {value: ƒ, writable: true, enumerable: false, configurable: true}
__defineGetter__: {value: ƒ, writable: true, enumerable: false, configurable: true}
__defineSetter__: {value: ƒ, writable: true, enumerable: false, configurable: true}
__lookupGetter__: {value: ƒ, writable: true, enumerable: false, configurable: true}
__lookupSetter__: {value: ƒ, writable: true, enumerable: false, configurable: true}
__proto__: Object
```


### Object.values(obj) (ES8)
Object.values() 方法与Object.keys类似。返回一个给定对象自己的所有可枚举属性值的数组，值的顺序与使用for...in循环的顺序相同 ( 区别在于for-in循环枚举原型链中的属性 )
```js
var obj = {a:1,b:2,c:3};
Object.keys(obj); // ['a','b','c']
Object.values(obj); // [1,2,3]
```

### Object.entries(obj) (ES8)
Object.entries() 方法返回一个给定对象自己的可枚举属性[key，value]对的数组，数组中键值对的排列顺序和使用 for...in 循环遍历该对象时返回的顺序一致（区别在于一个for-in循环也枚举原型链中的属性）
```js
var obj = {a:1,b:2,c:3};
Object.keys(obj); // ['a','b','c']
Object.values(obj); // [1,2,3]
Object.entries(obj); // [['a',1],['b',2],['c',3]]
```

## 小结
您可能会发现MDN上还有一些API，本文没有列举到。因为那些是非标准的API。熟悉对象的API对理解原型和原型链相关知识会有一定帮助。常用的API主要有Object.prototype.toString()，Object.prototype.hasOwnProperty()， Object.getPrototypeOf(obj)，Object.create()，Object.defineProperty，Object.keys(obj)，Object.assign()。


[原文](https://segmentfault.com/a/1190000010753942)