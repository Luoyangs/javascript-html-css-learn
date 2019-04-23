# 九个Console命令，让js调试更简单

## 显示信息的命令
```
console.log('hello');
console.info('信息');
console.error('错误');
console.warn('警告');

```

## 占位符
console上述的集中度支持printf的占位符格式，支持的占位符有：字符（%s）、整数（%d或%i）、浮点数（%f）和对象（%o）:

占位符  |	作用
:----------- | :-----------
%s | 字符串
%d or %i| 整数
%f| 浮点数
%o| 可展开的DOM
%O | 列出DOM的属性
%c | 根据提供的css样式格式化字符串

```
console.log("%d年%d月%d日",2011,3,26);

```

%o、%O都是用来输出Object对象的，对普通的Object对象，两者没区别，但是打印dom节点时就不一样了：
```js
// 格式成可展开的的DOM，像在开发者工具Element面板那样可展开 
console.log('%o',document.body.firstElementChild); 
// 像JS对象那样访问DOM元素，可查看DOM元素的属性 
// 等同于console.dir(document.body.firstElementChild) 
console.log('%O',document.body.firstElementChild);
```

%c占位符是最常用的。使用%c占位符时，对应的后面的参数必须是CSS语句，用来对输出内容进行CSS渲染。常见的输出方式有两种：文字样式、图片输出。
```js
console.log("%cHello world,欢迎您！","color: red; font-size: 20px"); 
//输出红色的、20px大小的字符串：Hello world,欢迎您！
```

![img](https://cloud.githubusercontent.com/assets/7871813/20181741/5156e57a-a79a-11e6-9f5d-d74908733fce.png)

由于 console不能定义img，因此用背景图片代替。此外，console不支持width和height，利用空格和font-size代替；还可以使用padding和line-height代替宽高。


## 信息分组
```js
console.group("第一组信息");
  console.log("第一组第一条:我的博客(http://www.ido321.com)");
　console.log("第一组第二条:CSDN(http://blog.csdn.net/u011043843)");
console.groupEnd();

console.group("第二组信息");
　console.log("第二组第一条:程序爱好者QQ群： 259280570");
  console.log("第二组第二条:欢迎你加入");
console.groupEnd();
```

![img](https://cloud.githubusercontent.com/assets/7871813/17443563/d824b86c-5b6d-11e6-83fa-e623693d3118.png)


## 查看对象的信息
```js
var info = {
  blog:"http://www.ido321.com",
  QQGroup:259280570,
  message:"程序爱好者欢迎你的加入"
};
console.dir(info);
```

![img](https://cloud.githubusercontent.com/assets/7871813/17443571/e6d04f34-5b6d-11e6-9ed0-6b64afd5587a.png)


## 显示某个节点的内容
```js
<!DOCTYPE html>
<html>
<head>
  <title>常用console命令</title>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>
<body>
  <div id="info">
    <h3>我的博客：www.ido321.com</h3>
    <p>程序爱好者:259280570,欢迎你的加入</p>
  </div>
  <script type="text/javascript">
    var info = document.getElementById('info');
    console.dirxml(info);
  </script>
</body>
</html>
```

![img](https://cloud.githubusercontent.com/assets/7871813/17443585/f6e7ce7e-5b6d-11e6-95b2-11b18e041a8a.png)


## 判断变量是否是真
console.assert()用来判断一个表达式或变量是否为真。如果结果为否，则在控制台输出一条相应信息，并且抛出一个异常。
```js
var result = 1;
console.assert( result );
var year = 2014;
console.assert(year == 2018 );
```

![img](https://cloud.githubusercontent.com/assets/7871813/17443601/0c202f34-5b6e-11e6-9b50-ce0cbc843ea5.png)


## 追踪函数的调用轨迹
console.trace()用来追踪函数的调用轨迹
```js
<script type="text/javascript">
    /*函数是如何被调用的，在其中加入console.trace()方法就可以了*/
　　
    function add(a,b){
      console.trace();
　　　　return a+b;
　　}
　　var x = add3(1,1);
　　function add3(a,b){return add2(a,b);}
　　function add2(a,b){return add1(a,b);}
　　function add1(a,b){return add(a,b);}
</script>
```
![img2](https://cloud.githubusercontent.com/assets/7871813/17443612/1b91bf50-5b6e-11e6-8bb8-2441435521bf.png)


## 计时功能
```js
<script type="text/javascript">
  console.time("控制台计时器一");
  for(var i=0;i<1000;i++){
  　　for(var j=0;j<1000;j++){}
  }
  console.timeEnd("控制台计时器一");
</script>
```

## console.profile()的性能分析
性能分析（Profiler）就是分析程序各个部分的运行时间，找出瓶颈所在，使用的方法是console.profile()。
```js
<script type="text/javascript">
　function All() {
    alert(11);
　　 for(var i=0;i<10;i++){
      funcA(1000);
     }
　　　funcB(10000);
　　}
   
  function funcA(count){
    for(var i=0;i<count;i++){}
  }

  function funcB(count){
    for(var i=0;i<count;i++){}
  }

  console.profile('性能分析器');
  All();
  console.profileEnd();
</script>
```

![img](https://cloud.githubusercontent.com/assets/7871813/17443637/438c79b4-5b6e-11e6-896a-5d9a0c5da63d.png)