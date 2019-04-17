# 写 React / Vue 项目时为什么要在组件中写 key，其作用是什么

没有绑定key的情况下，并且在遍历模板简单的情况下，会导致虚拟新旧节点对比更快，节点也会复用。而这种复用是就地复用，一种鸭子辩型的复用。以下为简单的例子:

```js
<div id="app">
    <div v-for="i in dataList">{{ i }}</div>
</div>

<script>
var vm = new Vue({
  el: '#app',
  data: {
    dataList: [1, 2, 3, 4, 5]
  }
});
</script>
```

以上的例子，v-for的内容会生成以下的dom节点数组，我们给每一个节点标记一个身份id：
```
[
  '<div>1</div>', // id： A
  '<div>2</div>', // id:  B
  '<div>3</div>', // id:  C
  '<div>4</div>', // id:  D
  '<div>5</div>'  // id:  E
]
```

改变dataList数据，进行数据位置替换，对比改变后的数据

```
 vm.dataList = [4, 1, 3, 5, 2] // 数据位置替换

// 没有key的情况， 节点位置不变，但是节点innerText内容更新了
[
  '<div>4</div>', // id： A
  '<div>1</div>', // id:  B
  '<div>3</div>', // id:  C
  '<div>5</div>', // id:  D
  '<div>2</div>'  // id:  E
]

// 有key的情况，dom节点位置进行了交换，但是内容没有更新
// <div v-for="i in dataList" :key='i'>{{ i }}</div>
[
  '<div>4</div>', // id： D
  '<div>1</div>', // id:  A
  '<div>3</div>', // id:  C
  '<div>5</div>', // id:  E
  '<div>2</div>'  // id:  B
]
```

增删dataList列表项
```
vm.dataList = [3, 4, 5, 6, 7] // 数据进行增删

// 1. 没有key的情况， 节点位置不变，内容也更新了
[
  '<div>3</div>', // id： A
  '<div>4</div>', // id:  B
  '<div>5</div>', // id:  C
  '<div>6</div>', // id:  D
  '<div>7</div>'  // id:  E
]

// 2. 有key的情况， 节点删除了 A, B 节点，新增了 F, G 节点
// <div v-for="i in dataList" :key='i'>{{ i }}</div>
[
  '<div>3</div>', // id： C
  '<div>4</div>', // id:  D
  '<div>5</div>', // id:  E
  '<div>6</div>', // id:  F
  '<div>7</div>'  // id:  G
]
```

从以上来看，不带有key，并且使用简单的模板，基于这个前提下，可以更有效的复用节点，diff速度来看也是不带key更加快速的，因为带key在增删节点上有耗时。这就是vue文档所说的默认模式。但是这个并不是key作用，而是没有key的情况下可以对节点就地复用，提高性能。

这种模式会带来一些隐藏的副作用，比如可能不会产生过渡效果，或者在某些节点有绑定数据（表单）状态，会出现状态错位。VUE文档也说明了 这个默认的模式是高效的，但是只适用于不依赖子组件状态或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出

楼下 @yeild 也提到，在不带key的情况下，对于简单列表页渲染来说diff节点更快是没有错误的。但是这并不是key的作用呀。

### 但是key的作用是什么？
> key是给每一个vnode的唯一id,可以依靠key,更准确, 更快的拿到oldVnode中对应的vnode节点。


1. 更准确

因为带key就不是就地复用了，在sameNode函数 a.key === b.key对比中可以避免就地复用的情况。所以会更加准确。

2. 更快

利用key的唯一性生成map对象来获取对应节点，比遍历方式更快。(这个观点，就是我最初的那个观点。从这个角度看，map会比遍历更快。)

vue和react都是采用diff算法来对比新旧虚拟节点，从而更新节点。在vue的diff函数中（建议先了解一下diff算法过程）。
在交叉对比中，当新节点跟旧节点头尾交叉对比没有结果时，会根据新节点的key去对比旧节点数组中的key，从而找到相应旧节点（这里对应的是一个key => index 的map映射）。如果没找到就认为是一个新增节点。而如果没有key，那么就会采用遍历查找的方式去找到对应的旧节点。一种一个map映射，另一种是遍历查找。相比而言。map映射的速度更快。
vue部分源码如下：
```js
// vue项目  src/core/vdom/patch.js  -488行
// 以下是为了阅读性进行格式化后的代码

// oldCh 是一个旧虚拟节点数组
if (isUndef(oldKeyToIdx)) {
  oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
}
if(isDef(newStartVnode.key)) {
  // map 方式获取
  idxInOld = oldKeyToIdx[newStartVnode.key]
} else {
  // 遍历方式获取
  idxInOld = findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
}

function createKeyToOldIdx (children, beginIdx, endIdx) {
  let i, key
  const map = {}
  for (i = beginIdx; i <= endIdx; ++i) {
    key = children[i].key
    if (isDef(key)) map[key] = i
  }
  return map
}

// sameVnode 是对比新旧节点是否相同的函数
function findIdxInOld (node, oldCh, start, end) {
  for (let i = start; i < end; i++) {
    const c = oldCh[i]
  
    if (isDef(c) && sameVnode(node, c)) return i
  }
}
```


