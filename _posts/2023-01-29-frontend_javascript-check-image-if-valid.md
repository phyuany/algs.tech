---
title: js怎样判断图片链接是否有效？
date: 2023-01-29 20:38:00 +0800
categories: [前端]
tags: []
pin: false
---

> 撰写时间：2020-08-03，整理时间：2023-01-29

要判断图片链接是否有效，我们首先创建一个Image对象，然后把链接设置在Image对象上，如果能正常加载，则代表链接有效。因为图片是异步加载的，所以我们需要Promise来处理。创建一个处理函数，如下代码：

```javascript
function checkImgExists(imgurl) {
    return new Promise(function(resolve, reject) {
      var ImgObj = new Image()
      ImgObj.src = imgurl
      ImgObj.onload = function(res) {
        resolve(res)
      }
      ImgObj.onerror = function(err) {
        reject(err)
      }
    })
}
```

根据以上代码，当图片加载成功时，resolve（完成）被触发，当图片加载失败时，reject（失败）被触发。于是可以使用一下代码进行调用测试

```javascript
checkImgExists('https://test.com/dssd=0.jpg').then(()=>{
    //success callback
    console.log('有效链接')
}).catch(()=>{
    //fail callback
    console.log('无效链接')
})
```

全部代码如下：

```js
function checkImgExists(imgurl) {
    return new Promise(function(resolve, reject) {
      var ImgObj = new Image();
      ImgObj.src = imgurl;
      ImgObj.onload = function(res) {
        resolve(res);
      }
      ImgObj.onerror = function(err) {
        reject(err)
      }
    })
}

checkImgExists('https://test.com/20200803115749u=2876792700,1627849181&fm=26&gp=0.jpg').then(()=>{
    //success callback
    console.log('有效链接')
}).catch(()=>{
    //fail callback
    console.log('无效链接')
})
```
