---
title: 微信小程序wx.getUserInfo接口获取用户信息失败，新版SDK怎样获取用户信息？
date: 2023-01-29 20:28:00 +0800
categories: [前端]
tags: []
pin: false
---

> 撰写时间：2019-12-10，整理时间：2023-01-29

## 一、简述

在微信小程序的官方文档中提到，调用`wx.getUserInfo`接口将能获取小程序用户的信息，接口返回的信息格式如下

```json
{
  "nickName": "Band",
  "gender": 1,
  "language": "zh_CN",
  "city": "Guangzhou",
  "province": "Guangdong",
  "country": "CN",
  "avatarUrl": "http://wx.qlogo.cn/mmopen/vi_32/1vZvI39NWFQ9XM4LtQpFrQJ1xlgZxx3w7bQxKARol6503Iuswjjn6nIGBiaycAjAtpujxyzYsrztuuICqIM5ibXQ/0"
}
```

不过，你会发现调用这个接口并没有返回用户信息。逛论坛才知道，这个接口被抛弃了。但是腾讯也提供了新的方式，以下演示获取用户信息的流程。

>参考链接: [https://developers.weixin.qq.com/community/develop/doc/000aee01f98fc0cbd4b6ce43b56c01](https://developers.weixin.qq.com/community/develop/doc/000aee01f98fc0cbd4b6ce43b56c01)
>

## 二、实现过程

### 2.1 使用button组件获取用户信息

使用一个button组建，并将 open-type 指定为 getUserInfo 类型，用户允许授权后，可获取用户基本信息。

```html
<button open-type="getUserInfo" lang="zh_CN" bindtap="onGotUserInfo" bindgetuserinfo="bindGetUserInfo">获取用户信息</button>
```

属性说明

- open-type

通过不同属性值，小程序会弹出一个不同的授权提示窗口，让用户选择是否授权。我们要获取用户信息，所以值指定的是getUserInfo，更多属性值可以参考微信小程序的[button组件文
档](https://developers.weixin.qq.com/miniprogram/dev/component/button.html)

- bindtap

绑定一个授权结果回调函数，在js文件中创建对应方法，详细代码如下

```javascript
onGotUserInfo: function(e) {
    console.log(e.detail.errMsg)
    console.log(e.detail.userInfo)
    console.log(e.detail.rawData)
  }
```

点击按钮之后，弹出授权窗口，如下图：

![01_meitu_1.jpg](/img/frontend/02-01.jpeg)

- bindgetuserinfo

可以从bindgetuserinfo回调中获取到用户信息，在这里我填写的是bindGetUserInfo，要在js文件中创建对应的方法，详细代码如下

```javascript
bindGetUserInfo: function(e) {
    console.log('回调成功')
    console.dir(e)
  }
```

上面回调方法中，我们打印返回的数据，结果如下截图所示：

![无标题.png](/img/frontend/02-02.pn)

可以看到，在回调信息中，包含的微信用户基本信息的键值对，如下示例：

```javascript
userInfo:
avatarUrl: "https://wx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTL8G6eBq1K7sk3CiceZ1vbY9IHO4kmJWQyFcEPc9trIGADva7ricI1ic7Zia8eWYAy4LLJwueejXOwKcg/132"
city: ""
country: "中国"
gender: 1
language: "zh_CN"
nickName: "碧海蓝天"
province: "贵州"
```

用户第一次点击按钮时弹出授权窗口，授权之后将不再弹出，bindtap回调方法第二次执行也不会再有返回数据。因此可以使用bindgetuserinfo回调方法获取用户信息，方便提交给业务服务器。

### 2.2 使用open-data展示用户基本信息

open-data控件用于展示微信开放的数据。不过这个控件只能显示数据，在写这篇博客的时候，微信是不支持读取open-data上的数据的，不知道将来会不会支持！
详细代码如下

```html
<view class="panel">
    <view>昵称：
      <open-data type="userNickName" lang="zh_CN"></open-data>
    </view>
    <view>头像：
      <open-data class="avatar" type="userAvatarUrl" lang="zh_CN"></open-data>
    </view>
    <view>性别：
      <open-data type="userGender" lang="zh_CN"></open-data>
    </view>
    <view>城市：
      <open-data type="userCity" lang="zh_CN"></open-data>
    </view>
    <view>省份：
      <open-data type="userProvince" lang="zh_CN"></open-data>
    </view>
    <view>国家：
      <open-data type="userCountry" lang="zh_CN"></open-data>
    </view>
    <view>语言：
      <open-data type="userLanguage" lang="zh_CN"></open-data>
    </view>
  </view>
```

运行项目后，看到效果如下图：

![02_meitu_2.jpg](/img/frontend/02-03.jpeg)

>在open-data标签中，最关键的属性是type,标志开放数据类型。详细说明可以参考[微信小程序open-data的开放文档](https://developers.weixin.qq.com/miniprogram/dev/component/open-data.html)
>

## 三、所有代码

以下是此文章涉及的所有代码

- index.wxml

```html
<!--index.wxml-->
<view class="container">
  <view class="panel">
    <button open-type="getUserInfo" lang="zh_CN" bindtap="onGotUserInfo" bindgetuserinfo="bindGetUserInfo">获取用户信息</button>
  </view>

  <view class="panel">
    <view>昵称：
      <open-data type="userNickName" lang="zh_CN"></open-data>
    </view>
    <view>头像：
      <open-data class="avatar" type="userAvatarUrl" lang="zh_CN"></open-data>
    </view>
    <view>性别：
      <open-data type="userGender" lang="zh_CN"></open-data>
    </view>
    <view>城市：
      <open-data type="userCity" lang="zh_CN"></open-data>
    </view>
    <view>省份：
      <open-data type="userProvince" lang="zh_CN"></open-data>
    </view>
    <view>国家：
      <open-data type="userCountry" lang="zh_CN"></open-data>
    </view>
    <view>语言：
      <open-data type="userLanguage" lang="zh_CN"></open-data>
    </view>
  </view>

</view>
```

- index.wxss

```css
/**index.wxss**/

.panel{
  margin-top: 20px;
}
```

- index.js

```javascript
//index.js
//获取应用实例
const app = getApp()

Page({
  onGotUserInfo: function(e) {
    console.log(e.detail.errMsg)
    console.log(e.detail.userInfo)
    console.log(e.detail.rawData)
  },

  bindGetUserInfo: function(e) {
    console.log('回调成功')
    console.dir(e)
  },
})
```

## 四、总结

微信小程序不再开放wx.getUserInfo接口，也就是说，我们要获取用户信息，需要引导用户手动点击一个button按钮弹出授权窗口，让用户手动授权，开发者才能成功获取用户信息。也许这样对开发者来说麻烦了点，不过，如果以用户的角度来看，这样确实比较安全。
