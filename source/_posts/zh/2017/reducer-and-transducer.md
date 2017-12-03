---
title: 优化数组的流式处理的性能 - Tranceducer 简介
date: 2017-12-03 12:44:40
categories: code
tags:
- functional
- reduce
- reducer
- transduce
- transducer
- 函数式
- js
---
很多人在开始熟练适用 JS 中的链式数组方法时都像打开了新世界。这是对函数式编程的初窥，而在使用如 `[].map(..)`, `[].filter(..)` 和 `[].reduce` 时，`reduce(..)` 必然的成为三者之间理解时间最长的函数。


传统的链式调用会出现如下的情况：
```javascript
const double = x => x * 2;
const addOne = x => x + 1;
const isSmartEnough = x => x >= 42;
const isNormalEnough = x => x <=100;
const sumReducer = (acc, item) => acc + item;

[4, 88, 41]
  .map(double)
  .map(addOne)
  .filter(isSmartEnough)
  .filter(isNormalEnough)
  .reduce(sumReducer);
```

我们知道，每一个调用都会对原列表做一次循环，这里的一个显而易见的优化是：

```javascript
[4, 88, 41]
  .map(compose(addOne, double))
// ...
```

通过对函数的组合，减少了循环的次数，看起来好多了。


但有些同学会思考，既然 `map` 可以被合并，那两个相同地 `filter` 能不能合并呢？就当前的方法来说，我们无法向 `map` 那样进行 `compose` 合并，因为每一个 `filterer` 的返回值都是 `boolean`。

### 船到桥头自然直

大家应该都试过用 `reduce` 来模拟 `map` 和 `filter` 的行为，而我们又希望能让 `filter` 和 `reduce` 能够合并，那么先将上面的两个 `fielterer` 转换成 `reducer` 来试一试

```javascript
const isSmartEnoughReducer = (list, item) => {
  if (isSmartEnough(item)) list.push(item);
  return list;
};
const isNormalEnoughReducer = (list, item) => {
  if (isNormalEnough(item)) list.push(item);
  return list;
};
```

等等，仔细一看，这俩货怎么长得这么像呢？进行进一步的抽象：

```javascript
const filterReducer = (predictFun) => {
  return function reducer(list, item) {
    if (predictFun(item)) list.push(item);
    return list;
  }
}

const isSmartEnoughReducer = filterReducer(isSmartEnough);
const isNormalEnoughReducer = filterReducer(isNormalEnough);
```

好多了。


但是总感觉 `list.push(item)` 这里不太符合纯函数的要求，如果把他提取出来呢？

```javascript
const combineListReducer = (list, item) => [...list, item];


const filterReducer = (predictFun, combineList) => {
  return function reducer(list, item) {
    if (predictFun(item)) return combineListReducer(list, item);
    return list;
  }
}

const isSmartEnoughReducer = filterReducer(isSmartEnough, combineListReducer);
const isNormalEnoughReducer = filterReducer(isNormalEnough, combineListReducer);

[4, 88, 41]
  .reduce(isSmartEnoughReducer)
  .reduce(isNormalEnoughReducer);
// ...
```

你问我为什么要叫 `combineListREDUCER`？因为一个 `reducer` 不就长这样吗？=)


整理整理上面的代码，思考一下，见证奇迹的时刻就要来了

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

看到上面的奇迹了吗？


没有？


因为奇迹在下面

### 奇迹来了

```javascript
const combineListReducer = (list, item) => [...list, item];

const filterReducer = (predictFun, combineList) => {
  return function reducer(list, item) {
    if (predictFun(item)) combineList(list, item);
    return list;
  }
}

const isSmartEnoughReducer = filterReducer(isSmartEnough, combineListReducer);
const isNormalAndSmartEnoughReducer = filterReducer(isNormalEnough, isSmartEnoughReducer);

[4, 88, 41]
  .reduce(isNormalAndSmartEnoughReducer, []);
  // => [88]
```

What? 合并了？


Wait for a minite.


What????


是的，你没错，一个 `reducer` 可以被当作另一个 `reducer` 的 `combineList` 函数，因为他们长得完全一样呀~


再来看看我们一开始的调用方法变成了什么样子
``` javascript
const sumReducer = (acc, item) => acc + item;

const combineListReducer = (list, item) => [...list, item];
const isSmartEnoughReducer = filterReducer(isSmartEnough, combineListReducer);
const isNormalAndSmartEnoughReducer = filterReducer(isNormalEnough, isSmartEnoughReducer);

[4, 88, 41]
  .map(compose(addOne, double))
  .reduce(isNormalAndSmartEnoughReducer, [])
  .reduce(sumReducer, 0);
```

请仔细观察 `combineListReducer` 和 `sumReducer`，你知道该怎样合并最后两句代码吗？

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

没错，`sumReducer` 和 `combineListReducer` 的功能是一样的！
扔掉 `combineListReducer` 吧！

```javascript
const sumReducer = (acc, item) => acc + item;

const isSmartEnoughReducer = filterReducer(isSmartEnough, sumReducer);
const isNormalAndSmartEnoughReducer = filterReducer(isNormalEnough, isSmartEnoughReducer);

[4, 88, 41]
  .map(compose(addOne, double))
  .reduce(isNormalAndSmartEnoughReducer, 0)
```

### 再进一步

几乎完成了我们的目标，现在消化一下上面的东西，我们一口气将 `map` 也合并进去：

```javascript
const mapReducer = (mapperFn, combineList) => {
  return function reducer(list,val){
    return combineList(list, mapperFn(val));
  };
}

const x = mapReducer(compose(addOne, double), sumReducer);
const y = filterReducer(isSmartEnough, x);
const z = filterReducer(isNormalEnough, y);

[4, 88, 41]
  .reduce(z, 0)
```

完成。

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

### 低级趣味

骗你的，你没发现这里还有优化的空间吗？？要不我干嘛还要写成 `x`, `y`, `z` ？？如果我们把 `mapReducer` 和 `filterReducer` (其实应该叫 `reducerGenerator`) 柯里化了呢？
```javascript
// (combineList) => (list, item) => list;
const xr = mapReducer(compose(addOne, double));
const yr = filterReducer(isSmartEnough);
const zr = filterReducer(isNormalEnough);

const composition = compose(
  zr,
  yr,
  xr,
);

const finalReducer = composition(sumReducer);
```

让我们更正式一点，顺便回到我们的标题

```javascript
// (combineList) => (list, item) => list;
const xr = mapReducer(compose(addOne, double));
const yr = filterReducer(isSmartEnough);
const zr = filterReducer(isNormalEnough);

// (combineList) => (list, item) => list;
const transducer = compose(
  zr,
  yr,
  xr,
);

[4, 88, 41]
  .reduce(transducer(sumReducer), 0);

// or

transduce(transducer, sumReducer, 0)([4, 88, 41]);
```

Fin.



其实还没完。

这段优化的代码里有一个很大的问题，你能找出来吗？



### 参考

 - [Function Light JS](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/apA.md)
 - [Transducers Explained](https://adispring.coding.me/2016/10/24/Transducers-Explained-Part-1/)
