---
title: 前端渲染大量数据处理方式  虚拟列表
date: 2022-2-22 16:48:58
comments: true #是否可评论
toc: true #是否显示文章目录
top_img: /img/sailing.jpg
categories: #分类
  - vue #这是一级分类
tags:
  - vue
---

### 什么是虚拟列表

`虚拟列表` 其实是按需显示的一种实现，即只对`可见区域`进行渲染，对`非可见区域`中的数据不渲染或`部分渲染`的技术，从而达到极高的渲染性能。

假设有 1 万条记录需要同时渲染，我们屏幕的`可见区域`的高度为 `500px`,而列表项的高度为 `50px`，则此时我们在屏幕中最多只能看到 10 个列表项，那么在首次渲染的时候，我们只需加载 10 条即可。

### 实现

虚拟列表的实现，实际上就是在首屏加载的时候，只加载`可视区域`内需要的列表项，当滚动发生时，动态通过计算获得`可视区域`内的列表项，并将`非可视区域`内存在的列表项删除。

- 计算当前可视区域起始数据索引(startIndex)
- 计算当前可视区域结束数据索引(endIndex)
- 计算当前可视区域的数据，并渲染到页面中
- 计算 startIndex 对应的数据在整个列表中的偏移位置 startOffset 并设置到列表上

由于只是对`可视区域`内的列表项进行渲染，所以为了保持列表容器的高度并可正常的触发滚动，将 Html 结构设计成如下结构：

```html
<div class="infinite-list-container">
  <div class="infinite-list-phantom"></div>
  <div class="infinite-list">
    <!-- item-1 -->
    <!-- item-2 -->
    <!-- ...... -->
    <!-- item-n -->
  </div>
</div>
```

- infinite-list-container 为可视区域的容器
- infinite-list-phantom 为容器内的占位，高度为总列表高度，用于形成滚动条
- infinite-list 为列表项的渲染区域

接着，监听 infinite-list-container 的 scroll 事件，获取滚动位置 scrollTop

- 假定可视区域高度固定，称之为 `screenHeight`
- 假定列表每项高度固定，称之为 `itemSize`
- 假定列表数据称之为 `listData`
- 假定当前滚动位置称之为 `scrollTop`

则可推算出：

- 列表总高度 `listHeight` = `listData`.length \* `itemSize`
- 可显示的列表项数 `visibleCount` = Math.ceil(`screenHeight` / `itemSize`)
- 数据的起始索引 `startIndex` = Math.floor(`scrollTop` / `itemSize`)
- 数据的结束索引 `endIndex` = `startIndex` + `visibleCount`
- 列表显示数据为 `visibleData` = `listData`.slice(`startIndex`,`endIndex`)

当滚动后，由于渲染区域相对于可视区域已经发生了偏移，此时我需要获取一个偏移量 startOffset，通过样式控制将渲染区域偏移至可视区域中。

- 偏移量 `startOffset` = scrollTop - (scrollTop % `itemSize`);

最终的简易代码如下：

```html
<template>
  <div ref="list" class="infinite-list-container" @scroll="scrollEvent($event)">
    <div
      class="infinite-list-phantom"
      :style="{ height: listHeight + 'px' }"
    ></div>
    <div class="infinite-list" :style="{ transform: getTransform }">
      <div
        ref="items"
        class="infinite-list-item"
        v-for="item in visibleData"
        :key="item.id"
        :style="{ height: itemSize + 'px', lineHeight: itemSize + 'px' }"
      >
        {{ item.value }}
      </div>
    </div>
  </div>
</template>
```

```js
export default {
  name: "VirtualList",
  props: {
    //所有列表数据
    listData: {
      type: Array,
      default: () => [],
    },
    //每项高度
    itemSize: {
      type: Number,
      default: 200,
    },
  },
  computed: {
    //列表总高度
    listHeight() {
      return this.listData.length * this.itemSize;
    },
    //可显示的列表项数
    visibleCount() {
      return Math.ceil(this.screenHeight / this.itemSize);
    },
    //偏移量对应的style
    getTransform() {
      return `translate3d(0,${this.startOffset}px,0)`;
    },
    //获取真实显示列表数据
    visibleData() {
      return this.listData.slice(
        this.start,
        Math.min(this.end, this.listData.length)
      );
    },
  },
  mounted() {
    this.screenHeight = this.$el.clientHeight;
    this.start = 0;
    this.end = this.start + this.visibleCount;
  },
  data() {
    return {
      //可视区域高度
      screenHeight: 0,
      //偏移量
      startOffset: 0,
      //起始索引
      start: 0,
      //结束索引
      end: null,
    };
  },
  methods: {
    scrollEvent() {
      //当前滚动位置
      let scrollTop = this.$refs.list.scrollTop;
      //此时的开始索引
      this.start = Math.floor(scrollTop / this.itemSize);
      //此时的结束索引
      this.end = this.start + this.visibleCount;
      //此时的偏移量
      this.startOffset = scrollTop - (scrollTop % this.itemSize);
    },
  },
};
```

[原作者 https://juejin.cn/post/6844903982742110216](https://juejin.cn/post/6844903982742110216)
