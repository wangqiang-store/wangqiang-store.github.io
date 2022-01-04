---
title: animejs实现走马灯效果
date: 2022-1-4 15:44:30
comments: true #是否可评论
toc: true #是否显示文章目录
top_img: /img/sailing.jpg
categories: #分类
  - vue #这是一级分类
tags:
  - vue
  - animejs
---

```javascript
 npm install animejs
```

> 在 main.js 中

```javascript
// 引入animejs
import anime from "animejs";
//注册全局方法
const app = createApp(App);
app.config.globalProperties.$anime = anime;
```

> vue 文件模板

```html
<div class="wrapper" @mousemove="handlePause()" @mouseleave="handlePlay">
  <div class="boxes">
    <div
      class="box"
      v-for="(item,index) in overdueCompanyList"
      :key="item.id"
    ></div>
  </div>
</div>
```

> js

```javascript
// 初始化走马灯
/*
*  overdueCompanyList 元素列表
*  40 元素高度
*  proxy vue实例 getCurrentInstance()
*/
initAnime: () => {
  // DATA.yTrans = [];
  proxy.$anime.set('.box', {
    translateY: function (el: any, i: number, l: any) {
    DATA.yTrans[i] = { y: i * 40 };
      return i * 40;
    },
  });
  DATA.animation = proxy.$anime({
    targets: DATA.yTrans,
    duration: ((DATA.overdueCompanyList.length * 40) / 20.8) * 1000, //走一周持续时间
    easing: 'linear',
    y: `-=${40 * DATA.overdueCompanyList.length}`,
    loop: true,
    update: function (anim: any) {
    proxy.$anime.set('.box', {
    translateY: function (el: any, i: number, l: any) {
      // 计算元素总高度与yTrans当前值取余 判断 >0? 从而计算 偏移距离
      return DATA.yTrans[i]?.y % (40 * DATA.overdueCompanyList.length) > 0
            ? DATA.yTrans[i]?.y % (40 * DATA.overdueCompanyList.length)
            : (DATA.yTrans[i]?.y % (40 * DATA.overdueCompanyList.length)) + 40 * DATA.overdueCompanyList.length;
          },
        });
      },
    });
  // animation.pause();
},
handlePlay: () => {
  // 鼠标移出 动画播放
  DATA.animation && DATA.animation.play();
},
handlePause: () => {
  // 鼠标移入 动画暂停
  DATA.animation && DATA.animation.pause();
}
  //移除动画
onUnmounted(() => {
  DATA.animation && DATA.animation.remove(DATA.yTrans);
});
```

> style

```scss
// 走马灯
.wrapper {
  width: 350px;
  height: 200px;
  position: absolute;
  // margin: 40px auto 0;
  background: #fff;
  overflow: hidden;
  // margin-bottom: 40px;
  right: 50px;
  bottom: 50px;
}
.box {
  width: 350px;
  height: 40px;
  position: absolute;
  cursor: pointer;
  background-color: #fff;
  font-size: 12px;
  color: #989898;
  text-align: center;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: nowrap;
}
.boxes {
  // position: relative;
  top: -40px;
}
```
