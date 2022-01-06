---
title: axios阻止重复请求
date: 2022-1-5 16:19:41
comments: true
toc: true
top_img: /img/sailing.jpg
categories:
  - axios
tags:
  - axios
---

## 前言

在实际项目中，我们可能需要对请求进行“防抖”处理。这里主要是为了阻止用户在某些情况下短时间内重复点击某个按钮，导致前端向后端重复发送多次请求。这里我列举两种比较常见的实际情况：

1. PC 端 - 用户双击搜索按钮，可能会触发两次搜索请求
2. 移动端 - 因移动端没有点击延迟，所以极易造成误操作或多操作，造成请求重发

## 核心——CancelToken

在 Axios 中取消请求最核心的方法是 CanelToken

> 注意设置是全局请求拦截,所以当有些请求让它重复请求的话 设置白名单

```js
// 拦截请求白名单
let whiteListUrls = [
  "/sysUser/checkRepeat",
  ...
];
```

```js
// 正在进行中的请求列表
let reqList: any[] = [];

/**
 * 阻止重复请求
 * @param {array} reqList - 请求缓存列表
 * @param {string} url - 当前请求地址
 * @param {function} cancel - 请求中断函数
 * @param {string} errorMessage - 请求中断时需要显示的错误信息
 */
const stopRepeatRequest = function (
  reqList: any,
  url: string,
  cancel: any,
  errorMessage?: string
) {
  const errorMsg = errorMessage || "";
  for (let i = 0; i < reqList.length; i++) {
    // 过滤白名单请求
    if (reqList[i] === url && !whiteListUrls.includes(url)) {
      cancel(errorMsg);
      return;
    }
  }
  reqList.push(url);
};

/**
 * 允许某个请求可以继续进行
 * @param {array} reqList 全部请求列表
 * @param {string} url 请求地址
 */
const allowRequest = function (reqList: any, url: string) {
  for (let i = 0; i < reqList.length; i++) {
    // 过滤白名单请求
    if (reqList[i] === url && !whiteListUrls.includes(url)) {
      reqList.splice(i, 1);
      break;
    }
  }
};
```

在 axios 请求拦截器中设置如下代码

```js
// request拦截器
service.interceptors.request.use(
  (config) => {
    if (store.getters.token) {
      config.headers["Authorization"] = getToken(); // 让每个请求携带自定义token 请根据实际情况自行修改 Accept
    }
    let cancel;
    // 设置cancelToken对象
    config.cancelToken = new axios.CancelToken(function(c) {
      cancel = c;
    });
    // 阻止重复请求。当上个请求未完成时，相同的请求不会进行
    stopRepeatRequest(reqList, config.url as string, cancel);
    return config;
  },
  (error) => {
    Promise.reject(error);
  }
);
```

在响应拦截器设置拦截的间隔

```js
service.interceptors.response.use(
  (response) => {
  let time = setTimeout(() => {
      allowRequest(reqList, response.config.url as string);
      clearTimeout(time);
    }, 200);
  })
```
