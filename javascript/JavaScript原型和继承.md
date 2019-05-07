# JavaScript原型和继承

### prototype属性
``Js所有的函数都有一个prototype属性``，这个属性引用了一个对象，即原型对象，也简称原型。这个函数包括构造函数和普通函数，我们讲的更多是构造函数的原型，但是也不能否定普通函数也有原型。譬如普通函数
```js
function F(){};
alert(F.prototype instanceof Object);//true
```

默认情况下，原型对象也会获得一个constructor属性，该属性包含一个指针，指向prototype属性所在的函数
```js
Person.prototype.constructor === Person
```
在面向对象的语言中，我们使用类来创建一个自定义对象。然而js中所有事物都是对象，那么用什么办法来创建自定义对象呢？这就需要用到js的原型：

我们可以简单的把prototype看做是一个模版，新创建的自定义对象都是这个模版（prototype）的一个拷贝 （实际上不是拷贝而是链接，只不过这种链接是不可见，新实例化的对象内部有一个看不见的__Proto__指针，指向原型对象）。

### 关于__proto__
__proto__是对[[propertyName]]属性的实现（``是只能对象可以拥有的属性，并且是不可访问的内部属性``），它指向对象的构造函数的原型对象。如下：
```js
function Person(name) {
  this.name = name;
}
var p1 = new Person();
p1.__proto__ === Person.prototype;//true
```
__proto__只是浏览器的私有实现，目前ECMAScript标准实现方法是Object.getPrototypeOf(object)
```js
function Person(name) {
  this.name = name;
}
var p1 = new Person();
Object.getPrototypeOf(p1) === Person.prototype;//true
```

### 原型对象
``原型对象初始是空的``，也就是没有一个成员（即原型属性和原型方法）。
```js
function A(x){
  this.x=x;
}

var num = 0;
for(o in A.prototype){
  alert(o);// alert 原型属性名字
  num++;
} 
alert(num);//0
```
但是，一旦定义了原型属性或原型方法，则所有通过该构造函数实例化出来的所有对象，都继承了这些原型属性和原型方法，这是通过内部的__proto__链来实现的。
```js
A.prototype.say=function(){alert('haha');}
```
那所有的A的对象都具有了say方法，这个原型对象的say方法是唯一的副本给大家共享的，而不是每一个对象都有关于say方法的一个副本。

## 原型与继承
由于js不像java那样是真正面向对象的语言，js是基于对象的，它没有类的概念。所以，要想实现继承，可以通过构造函数和原型的方式模拟实现类的功能。
* 类式继承（构造函数间的继承）  
* 原型链继承（对象间的继承）

### 类式继承详解
类式继承是在子类型构造函数的内部调用超类型的构造函数。
```js
function A(x){
  this.x = x;
}
function B(x,y){
  // 以下实现类似A.call(this, x)的效果
  this.tempObj = A;
  this.tempObj(x);
  delete this.tempObj;
  this.y = y;
}
```
第5、6、7行：创建临时属性tmpObj引用构造函数A，然后在B内部执行（注意，这里执行函数的时候并没有用new），执行完后删除。当在B内部执行了this.x=x后（这里的this是B的对象），B当然就拥有了x属性，当然B的x属性和A的x属性两者是独立，所以并不能算严格的继承。第5、6、7行有更简单的实现，就是通过call(apply)方法：A.call(this,x);

这两种方法都有将this传递到A的执行里，this指向的是B的对象，这就是为什么不直接A(x)的原因。这种继承方式即是类继承（js没有类，这里只是指构造函数），虽然继承了A构造对象的所有属性方法，但是不能继承A的原型对象的成员。而要实现这个目的，就是在此基础上再添加原型继承。

### 原型链继承详解
原型式继承是借助已有的对象创建新的对象，将子类的原型指向父类，就相当于加入了父类这条原型链。
```js
function A(x){
  this.x = x;
}
A.prototype.a = 'a';
function B(x,y){
  A.call(this,x);
  this.y = y;
}
B.prototype.b1 = function(){
  alert('b1');
}
B.prototype = new A();
B.prototype.b2 = function(){
  alert('b2');
}
B.prototype.constructor = B;
var obj = new B(1,3);
```
这个例子讲的就是B继承A。第7行类继承：A.call(this.x);上面已讲过。实现原型继承的是第12行：B.prototype = new A();

就是说把B的原型指向了A的1个实例对象，这个实例对象具有x属性，为undefined，还具有a属性，值为"a"。所以B原型也具有了这2个属性（或者说，B和A建立了原型链，B是A的下级）。而因为方才的类继承，B的实例对象也具有了x属性，也就是说obj对象有2个同名的x属性，此时原型属性x要让位于实例对象属性x，所以obj.x是1，而非undefined。第13行又定义了原型方法b2，所以B原型也具有了b2。

虽然第9~11行设置了原型方法b1，但是你会发现第12行执行后，B原型不再具有b1方法，也就是obj.b1是undefined。因为第12行使得B原型指向改变，原来具有b1的原型对象被抛弃，自然就没有b1了。

第12行执行完后，B原型（B.prototype）指向了A的实例对象，而A的实例对象的构造器是构造函数A，所以B.prototype.constructor就是构造对象A了（换句话说，A构造了B的原型）。alert( B.prototype.constructor )出来后就是"function A(x){...}" 。同样地，obj.constructor也是A构造对象，alert(obj.constructor)出来后就是"function A(x){...}" ，也就是说B.prototype.constructor === obj.constructor（true），但是B.prototype === obj.constructor.prototype（false），因为前者是B的原型，具有成员：x,a,b2，后者是A的原型，具有成员：a。如何修正这个问题呢，就在第16行，将B原型的构造器重新指向了B构造函数，那么B.prototype === obj.constructor.prototype（true），都具有成员：x,a,b2.

如果没有第16行，那是不是obj = new B(1,3)会去调用A构造函数实例化呢？答案是否定的，你会发现obj.y=3，所以仍然是调用的B构造函数实例化的。虽然obj.constructor===A(true)，但是对于new B()的行为来说，执行了上面所说的通过构造函数创建实例对象的3个步骤
* 第一步，创建空对象；
* 第二步，obj.__proto__ === B.prototype，B.prototype是具有x,a,b2成员的，obj.constructor指向了B.prototype.constructor，即构造函数A；
* 第三步，调用的构造函数B去设置和初始化成员，具有了属性x,y。

虽然不加16行不影响obj的属性，但如上一段说，却影响obj.constructor和obj.constructor.prototype。所以在使用了原型继承后，要进行修正的操作。

关于第12、16行，总言之，第12行使得B原型继承了A的原型对象的所有成员，但是也使得B的实例对象的构造器的原型指向了A原型，所以要通过第16行修正这个缺陷

## 继承的6种方法

### 1.原型链继承
为了让子类继承父类的属性（也包括方法），首先需要定义一个构造函数。然后，将父类的新实例赋值给构造函数的原型。
```js
function Parent(){
  this.name = 'mike';
}

function Child(){
  this.age = 12;
}
Child.prototype = new Parent();//Child继承Parent，通过原型，形成链条

var child = new Child();
alert(child.age);
alert(child.name);//得到被继承的属性
//继续原型链继承
function Brother(){   //brother构造
  this.weight = 60;
}
Brother.prototype = new Child();//继续原型链继承
var brother = new Brother();
alert(brother.name);//继承了Parent和Child,弹出mike
alert(brother.age);//弹出12
```
以上原型链继承还缺少一环，那就是Object，所有的构造函数都继承自Object。而继承Object是自动完成的，并不需要我们自己手动继承，那么他们的从属关系可以使用操作符instanceof和函数isPrototypeOf()判断，如下：
```js
alert(child instanceof Parent);//true
alert(child instanceof Child);//true
alert(brother instanceof Parent);//true
alert(brother instanceof Child);//true

alert(Parent.prototype.isPrototypeOf(child));//true
alert(Child.prototype.isPrototypeOf(child));//true
alert(Parent.prototype.isPrototypeOf(brother));//true
alert(Child.prototype.isPrototypeof(brother));//true
```
缺陷：
* 字面量重写原型会中断关系
* 使用引用类型的原型
* 并且子类型还无法给超类型传递参数。


### 2.构造函数继承
```js
function Parent(firstname) {
  this.fname=firstname;
  this.age=40;
  this.sayAge = function() {
    console.log(this.age);
  }
}
function Child(firstname) {
  this.parent=Parent;
  this.parent(firstname);
  delete this.parent;//以上三行也可以用call和apply函数改写
  this.saySomeThing = function() {
    console.log(this.fname);
    this.sayAge();
  }
}
var child=new  Child("李");
child.saySomeThing();

// call实现的方式
function Parent(firstname) {
  this.fname=firstname;
  this.age=40;
  this.sayAge=function() {
    console.log(this.age);
  }
}
Parent.prototype.say = function() {
  console.log('parent say');
}

function Child1(firstname) {
  this.saySomeThing1 = function() {
    console.log(this.fname);
    this.sayAge();
  }
  this.getName = function() {
    return firstname;
  }
}

function Child2(firstname) {
  this.saySomeThing2 = function() {
    console.log(this.fname);
    this.sayAge();
  }
  this.getName = function() {
    return firstname;
  }
}

var child1 = new Child1("张1");
Parent.call(child1,child1.getName());
child1.saySomeThing1();
child1.say();// TypeError: child1.say is not a function


var child2 = new Child2("张2");
Parent.call(child2,child2.getName());
child2.saySomeThing2();
```
优点：
* 这种方式可以实现多继承
* 也可以向父类传递参数

缺陷：
* 类式继承没有原型，子类型只是继承了父类型构造对象的属性和方法，没有继承父类型原型对象的成员。


### 3.组合继承
组合继承利用原型链+借用构造函数的模式解决了原型链继承和类式继承的问题。
```js
function Parent(age){
  this.name = ['mike','jack','smith'];
  this.age = age;
}
Parent.prototype.say = function(){
  return this.name + 'are both' + this.age;
}
function Child(age){
  Parent.call(this,age); // 第二次调用
}
Child.prototype = new Parent(); // 第一次调用
var child = new Child(1);
child.say(); // mike,jack,smith are both 1 
```
组合式继承是比较常用的一种继承方法，其背后的思路是 使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又保证每个实例都有它自己的属性。

缺陷：
* 组合继承的父类型在使用过程中会被调用两次；一次是创建子类型的时候，另一次是在子类型构造函数的内部。


### 4.原型式继承
```js
function obj(o){
  function F(){}
  F.prototype = o;
  return new F();
}
var person = {
  name : 'ss',
  arr : ['hh','kk','ll']
}

var b1 = obj(person);
b1.name;//ss
b1.name = 'join';
b1.name;//join

b1.arr;//hh,kk,ll
b1.arr.push('gg');
b1.arr;//hh,kk,ll,gg

var b2 = obj(person);
b2.name;//ss
b2.arr;//hh,kk,ll,gg
```
这里的b1和b2会共享原型对象person的属性

### 5.寄生式继承
这种继承方式是把原型式+工厂模式结合起来，目的是为了封装创建的过程。
```js
function obj(o){
  function F(){}
  F.prototype = o;
  return new F()
}

function create(o){
  var f = obj(o);
  f.say = function() {
    return this.arr
  }
  return f
}
```

### 6.寄生组合继承
```js
//通过这个函数，实现了原型链继承，但child只含有parent原型链中的属性
function create(parent,child){
  function F() {};
  F.prototype = parent.prototype;
  var f = new F();
  f.constructor = child;
  child.prototype = f;
}

function Parent(name){
  this.name = name;
  this.arr = ['heheh','guagua','jiji'];
}

Parent.prototype.say = function(){
  return this.name
}

function Child(name,age){
  Parent.call(this,name);//类式继承，这里继承parent构造函数中定义的属性
  this.age = age;
}

create(Parent,Child);

var child = new Child('trigkit4',21);
child.arr.push('nephew');
var child2 = new Child('jack',22);

child.arr;//
child.run();//只共享了方法
child2.arr;//引用问题解决
```
