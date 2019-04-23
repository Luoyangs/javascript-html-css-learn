# 模拟实现JS的new操作符

```js
new Vue({
    el: '#app',
    mounted(){},
});
```

new 做了什么?

```js
// 例子1
function Student(){
}
var student = new Student();
console.log(student); // {}
// student 是一个对象。
console.log(Object.prototype.toString.call(student)); // [object Object]
// 我们知道平时声明对象也可以用new Object(); 只是看起来更复杂
// 顺便提一下 `new Object`(不推荐)和Object()也是一样的效果
// 可以猜测内部做了一次判断，用new调用
/** if (!(this instanceof Object)) {
*    return new Object();
*  }
*/
var obj = new Object();
console.log(obj) // {}
console.log(Object.prototype.toString.call(obj)); // [object Object]

typeof Student === 'function' // true
typeof Object === 'function' // true
```

从这里例子中，我们可以看出：一个函数用new操作符来调用后，生成了一个全新的对象。而且Student和Object都是函数，只不过Student是我们自定义的，Object是JS本身就内置的。 再来看下控制台输出图，感兴趣的读者可以在控制台试试。

与new Object() 生成的对象不同的是new Student()生成的对象中间还嵌套了一层__proto__，它的constructor是Student这个函数。

```
// 也就是说：
student.constructor === Student;
Student.prototype.constructor === Student;
```

### 小结1：从这个简单例子来看，new操作符做了两件事：
* 创建了一个全新的对象。
* 这个对象会被执行[[Prototype]]（也就是__proto__）链接。

```js
// 例子2
function Student(name){
    console.log('赋值前-this', this); // {}
    this.name = name;
    console.log('赋值后-this', this); // {name: '轩辕Rowboat'}
}
var student = new Student('轩辕Rowboat');
console.log(student); // {name: '轩辕Rowboat'}
student.__proto__.constructor === Student; // true
student.constructor === Student; // true
Student.prototype.constructor === Student; // true
```

由此可以看出：这里Student函数中的this指向new Student()生成的对象student。同时也进一步证实了小结1，即：这个新创建的对象会被执行Student方法（student.__proto__）

### 小结2：从这个例子来看，new操作符又做了一件事：
生成的新对象会绑定到函数调用的this。

```js
// 例子3
function Student(name){
    this.name = name;
    // this.doSth();
}
Student.prototype.doSth = function() {
    console.log(this.name);
};
var student1 = new Student('轩辕');
var student2 = new Student('Rowboat');
console.log(student1, student1.doSth()); // {name: '轩辕'} '轩辕'
console.log(student2, student2.doSth()); // {name: 'Rowboat'} 'Rowboat'
student1.__proto__ === Student.prototype; // true
student2.__proto__ === Student.prototype; // true
// __proto__ 是浏览器实现的查看原型方案。
// 用ES5 则是：
Object.getPrototypeOf(student1) === Student.prototype; // true
Object.getPrototypeOf(student2) === Student.prototype; // true
```

![img](https://user-gold-cdn.xitu.io/2018/11/4/166dee0c1854fab5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 小结3：这个例子3再一次验证了小结1中的第2点。也就是这个对象会被执行[[Prototype]]（也就是__proto__）链接。``并且通过new Student()创建的每个对象将最终被[[Prototype]]链接到这个Student.protytype对象上。``

```js
// 例子4
function Student(name){
    this.name = name;
    // Null（空） null
    // Undefined（未定义） undefined
    // Number（数字） 1
    // String（字符串）'1'
    // Boolean（布尔） true
    // Symbol（符号）（第六版新增） symbol
    
    // Object（对象） {}
        // Function（函数） function(){}
        // Array（数组） []
        // Date（日期） new Date()
        // RegExp（正则表达式）/a/
        // Error （错误） new Error() 
    // return /a/;
}
var student = new Student('轩辕Rowboat');
console.log(student); {name: '轩辕Rowboat'}
```

前面六种基本类型都会正常返回{name: '轩辕Rowboat'}，后面的Object(包含Functoin, Array, Date, RegExg, Error)都会直接返回这些值。

### 小结4：
如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用会自动返回这个新的对象。


## 结合这些小结，整理在一起就是：

* 创建了一个全新的对象。
* 这个对象会被执行[[Prototype]]（也就是__proto__）链接(这条和下面第四条合并理解)。
* 生成的新对象会绑定到函数调用的this。
* 通过new创建的每个对象将最终被[[Prototype]]链接到这个函数的prototype对象上。
* 如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用会自动返回这个新的对象。


知道了这些现象，我们就可以模拟实现new操作符。直接贴出代码和注释
```js
/**
 * 模拟实现 new 操作符
 * @param  {Function} ctor [构造函数]
 * @return {Object|Function|Regex|Date|Error}      [返回结果]
 */
function newOperator(ctor){
  if(typeof ctor !== 'function'){
    throw 'newOperator function the first param must be a function';
  }
  // ES6 new.target 是指向构造函数
  newOperator.target = ctor;
  // 1.创建一个全新的对象，
  // 2.并且执行[[Prototype]]链接
  // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
  var newObj = Object.create(ctor.prototype);
  // ES5 arguments转成数组 当然也可以用ES6 [...arguments], Aarry.from(arguments);
  // 除去ctor构造函数的其余参数
  var argsArr = [].slice.call(arguments, 1);
  // 3.生成的新对象会绑定到函数调用的`this`。
  // 获取到ctor函数返回结果
  var ctorReturnResult = ctor.apply(newObj, argsArr);
  // 小结4 中这些类型中合并起来只有Object和Function两种类型 typeof null 也是'object'所以要不等于null，排除null
  var isObject = typeof ctorReturnResult === 'object' && ctorReturnResult !== null;
  var isFunction = typeof ctorReturnResult === 'function';
  if(isObject || isFunction){
      return ctorReturnResult;
  }
  // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，那么`new`表达式中的函数调用会自动返回这个新的对象。
  return newObj;
}

// 例子3 多加一个参数
function Student(name, age){
    this.name = name;
    this.age = age;
    // this.doSth();
    // return Error();
}
Student.prototype.doSth = function() {
    console.log(this.name);
};
var student1 = newOperator(Student, '轩辕', 18);
var student2 = newOperator(Student, 'Rowboat', 18);
// var student1 = new Student('轩辕');
// var student2 = new Student('Rowboat');
console.log(student1, student1.doSth()); // {name: '轩辕'} '轩辕'
console.log(student2, student2.doSth()); // {name: 'Rowboat'} 'Rowboat'

student1.__proto__ === Student.prototype; // true
student2.__proto__ === Student.prototype; // true
// __proto__ 是浏览器实现的查看原型方案。
// 用ES5 则是：
Object.getPrototypeOf(student1) === Student.prototype; // true
Object.getPrototypeOf(student2) === Student.prototype; // true
```

### Object.create() 用法举例
Object.create(proto, [propertiesObject]) 方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。 它接收两个参数，不过第二个可选参数是属性描述符（不常用，默认是undefined）。
```js
var anotherObject = {
    name: '轩辕Rowboat'
};
var myObject = Object.create(anotherObject, {
    age: {
        value：18,
    },
});
// 获得它的原型
Object.getPrototypeOf(anotherObject) === Object.prototype; // true 说明anotherObject的原型是Object.prototype
Object.getPrototypeOf(myObject); // {name: "轩辕Rowboat"} // 说明myObject的原型是{name: "轩辕Rowboat"}
myObject.hasOwnProperty('name'); // false; 说明name是原型上的。
myObject.hasOwnProperty('age'); // true 说明age是自身的
myObject.name; // '轩辕Rowboat'
myObject.age; // 18;

```

```js
if (typeof Object.create !== "function") {
    Object.create = function (proto, propertiesObject) {
        if (typeof proto !== 'object' && typeof proto !== 'function') {
            throw new TypeError('Object prototype may only be an Object: ' + proto);
        } else if (proto === null) {
            throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
        }

        if (typeof propertiesObject != 'undefined') throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");

        function F() {}
        F.prototype = proto;
        return new F();
    };
}

```



链接：https://juejin.im/post/5bde7c926fb9a049f66b8b52