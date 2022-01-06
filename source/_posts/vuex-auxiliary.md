---
title: vue3.0中使用vuex辅助函数
date: 2022-1-4 16:36:43
comments: true
toc: true
top_img: /img/sailing.jpg
categories:
  - vue
tags:
  - vue3.0
  - vuex
---

> 自定义 hooks

```js
import { computed } from "vue";
import {
  mapGetters,
  mapMutations,
  mapState,
  useStore,
  createNamespacedHelpers,
  mapActions,
} from "vuex";
/**
 * @param mapper  Array | Object
 * @param mapFn   Function
 * @param iscomputed 判断是否转变计算属性
 * @returns
 */
const useMapper = (
  mapper: Array<any> | Object,
  mapFn: Function,
  iscomputed: boolean = true
) => {
  const store = useStore(); //使用辅助函数解析成一个对象
  const storeStateFns = mapFn(mapper);
  const storeState: any = {}; //通过Object.keys拿到对象的所有key值，遍历，取出对应的value值，也就是函数
  Object.keys(storeStateFns).forEach((keyFn) => {
    // 这我们知道辅助函数的内部是通过this.$store来实现的
    // setup中没有this， 所以通过bind来改变this的指向
    const fn = storeStateFns[keyFn].bind({ $store: store }); //拿到函数，作为计算属性的参数，最后在留在一个对象中
    storeState[keyFn] = iscomputed ? computed(fn) : fn;
  }); // storeState是一个对象， key是字符串， value值是ref对象
  return storeState;
};

export const useState = (mapper: Array<any> | Object) => {
  return useMapper(mapper, mapState);
};

export const useMutation = (mapper: Array<any> | Object) => {
  return useMapper(mapper, mapMutations, false);
};

export const useAction = (mapper: Array<any> | Object) => {
  return useMapper(mapper, mapActions, false);
};

export const useGetters = (mapper: Array<any> | Object) => {
  return useMapper(mapper, mapGetters);
};

// vuex模块 传对应模块名称
export const useModuleState = (
  moduleName: string,
  mapper: Array<any> | Object
) => {
  let mapperFn = mapState;
  if (typeof moduleName === "string" && moduleName.length > 0) {
    mapperFn = createNamespacedHelpers(moduleName).mapState;
  } else {
    mapper = moduleName;
  }
  return useMapper(mapper, mapperFn);
};

export const useModuleGetters = (
  moduleName: string,
  mapper: Array<any> | Object
) => {
  let mapperFn = mapGetters;
  if (typeof moduleName === "string" && moduleName.length > 0) {
    mapperFn = createNamespacedHelpers(moduleName).mapGetters;
  } else {
    mapper = moduleName;
  }
  return useMapper(mapper, mapperFn);
};
```
