---
title: 前端面试 - js篇
date: 2022-11-22 09:45:58
top_img: /img/sailing.jpg
comments: true
toc: true
categories:
  - javascript
tags:
  - javascript
---

### 深克隆

```js
function deepClone(obj, map) {
  // 判断数据类型是否为引用类型 否返回数据本身
  if (typeof obj !== "object") obj;
  // 判断map是否包含 obj作为key的值 是返回 值
  if (map.get(obj)) map.get(obj);
  // 定义返回值
  let result = {};
  // 判断是否为数组
  if (
    Array.isArray(obj) ||
    Object.prototype.toString.call(obj) === "[object Array]"
  ) {
    let result = [];
  }
  map.set(obj, result);
  // 遍历对象
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      result[key] = deepClone(obj[key], map);
    }
  }
  return result;
}
```

### 防抖

```js
function debounce(fn, delay) {
  let time = null;
  return function () {
    let _that = this;
    let args = arguments;
    if (time) {
      clearTimeout(time);
    }
    time = setTimeout(() => {
      fn.apple(_that, args);
    }, delay);
  };
}
```

### 节流

```js
function debounce(fn, delay) {
  let time = null;
  return function () {
    let _that = this;
    let args = arguments;
    if (!time) {
      time = setTimeout(() => {
        fn.apple(_that, args);
        time = null;
      }, delay);
    }
  };
}
```

### 去重

```ts
// 去重
function delduplication(data: Array<any>) {
  // ES6
  return [...new Set(data)];

  // ES5
  return data.filter((item, index) => {
    return data.indexOf(item) === index;
  });
}
```

### 数组扁平化

```ts
// 数组扁平化
function flat(data: Array<any>) {
  // ES6
  return data.flat(Infinity);
  // ES5
  return data.reduce((init, cur) => {
    return init.concat(Array.isArray(cur) ? flat(cur) : cur);
  }, []);
}
```

### map

```ts
// 原生js实现map
function myMap(fn: Function, context: any) {
  let data = Array.prototype.slice.call(this);
  let result: Array<any> = [];
  for (let i = 0; i < data.length; i++) {
    result.push(fn.call(context, data[i], i, this));
  }
  return result;
}
```

### reduce

```ts
// 原生js实现reduce
function myReduce<T>(fn: Function, initData: T) {
  let data = Array.prototype.slice.call(this);
  let res = initData ? initData : data[0];
  let initIndex = initData ? 0 : 1;
  for (let i = initIndex; i < data.length; i++) {
    res = fn.call(null, res, data[i], this);
  }
  return res;
}
```

### call/apply

```js
// 原生js实现call/apply  实现apply只要把下一行中的...args换成args即可
function myCall(context = window, ...args) {
  let func = this;
  let fn = Symbol("fn");
  context[fn] = func;

  let res = context[fn](...args);

  delete context[fn];
  return res;
}
```

### 实现 Object.create 方法(常用)

```js
// 实现Object.create方法(常用)
function create(initData) {
  function F() {}
  F.prototype = initData;
  F.prototype.constructor = F;

  return new F();
}
```

### bind

```js
// 手写bing
function myBind(context, ...args) {
  let self = this;
  let found = function () {
    return self.apply(
      this instanceof found ? this : context || window,
      args.concat(Array.prototype.slice.call(arguments))
    );
  };
  found.prototype = Object.create(this.prototype);
  return found;
}
```

### new

```js
// 手写new方法
function myNew(fn, ...args) {
  let obj = Object.create(fn.prototype);
  let instance = fn.apply(obj, args);
  return typeof instance === "object" ? instance : obj;
}
```

### instanceof

```js
// 手写instanceof
function myInstanceof(left, right) {
  let proto = Object.getPrototypeOf(left);
  while (true) {
    if (proto === null) return false;
    if (proto === right.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
}
```

### promise

```js
// 手写promise  三种状态 pending fulfilled rejected
class myPromise {
  value: undefined;
  state: string;
  resolveArr: Function[];
  rejectArr: Function[];
  constructor(asyncBackFn) {
    this.value = undefined;
    this.state = "pending";
    this.resolveArr = [];
    this.rejectArr = [];

    let resolveFn = (res) => {
      if (this.state !== "pending") return;
      let timer = setTimeout(() => {
        this.state = "fulfilled";
        this.value = res;
        this.resolveArr.forEach((item) => item(this.value));
      }, 0);
    };

    let rejectFn = (res) => {
      if (this.state !== "pending") return;
      let timer = setTimeout(() => {
        this.state = "rejected";
        this.value = res;
        this.resolveArr.forEach((item) => item(this.value));
      }, 0);
    };

    try {
      //执行这个异步函数
      asyncBackFn(resolveFn, rejectFn);
    } catch (err) {
      //=>有异常信息按照rejected状态处理
      rejectFn(err);
    }
  }

  // 完成链式操作
  then(resolveCallback, rejectCallback) {
    typeof resolveCallback === "function"
      ? (resolveCallback = (result) => result)
      : null;
    typeof rejectCallback === "function"
      ? (rejectCallback = (error) => {
          throw new Error(error instanceof Error ? error.message : error);
        })
      : null;

    return new Promise((resolve, reject) => {
      this.resolveArr.push(() => {
        try {
          let x = resolveCallback(this.value);
          x instanceof Promise ? x.then(resolve, reject) : resolve(x);
        } catch (error) {
          reject(error);
        }
      });

      this.rejectArr.push(() => {
        try {
          let x = rejectCallback(this.value);
          x instanceof Promise ? x.then(resolve, reject) : resolve(x);
        } catch (error) {
          reject(error);
        }
      });
    });
  }
}
```

### 算法

```ts
// 算法
// 实现 pow(x, n) ，即计算 x 的整数 n 次幂函数（即，xn ）
function pow(x: number, n: number) {
  let copyx = x;
  // 判断 幂n 是正数 还是负数 或 0
  if (n > 0) {
    for (let i = 1; i < n; i++) {
      x = copyx * x;
    }
    return x;
  } else if (n == 0) {
    return 1;
  } else {
    n = n * -1;
    for (let i = 1; i < n; i++) {
      x = copyx * x;
    }
    return 1 / x;
  }
}

// 给你一个非负整数 x ，计算并返回 x 的 算术平方根 。
// 由于返回类型是整数，结果只保留 整数部分 ，小数部分将被 舍去 。
function mySqrt(x: number, i: number = 1): number {
  while (true) {
    if (i * i > x) return i - 1;
    i++;
  }
}

// 给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。
function search(nums: number[], target: number): number {
  let result: number = -1;
  for (let index = 0; index < nums.length; index++) {
    if (nums[index] === target) {
      result = index;
    }
  }
  return result;
}

// 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
// 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
// 你可以按任意顺序返回答案。
function twoSum(nums: number[], target: number): number[] {
  let result: number[] = [];
  nums.forEach((item, index) => {
    nums.forEach((key, nub) => {
      if (index !== nub && item + key === target) {
        result = [index, nub];
      }
    });
  });
  return result;
}
```
