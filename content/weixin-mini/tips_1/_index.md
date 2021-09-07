---
layout: single
title:  "微信小程序开发Tips_1"
date:   2021-08-30
categories: 微信小程序开发
toc: true
---
自定义组件加载慢 和 wx.switchTab 无法传递参数的问题

#### 问题1

- Page页面使用了自定义组件，自定义组件中又使用了Vant的Popup组件，导致Page启动时非常缓慢。如果不提前加载自定义组件，后面点击弹出的时候就会很缓慢。

> ##### 解决办法 
> - 可以给Vant Popup 外面嵌套一个 View 运用wx:if 控制这个View合适加载，在onReady方法中，启动嵌套的这个View

> - 在WXML中给自定义组件嵌套一个View 用wx:if 控制合适加载

```html
        <view wx:if="{{filter_loading}}">
            <filter id="filter" show="{{filter_show}}">
            </filter>
        </view>
```

> - 在js的onReady中启用这个View的加载

```javascript
      onReady: function () {
        this.setData({filter_loading: true})
      }
```

#### 问题2
- 小程序控制切换底部Tabbar的时候，有时候我们想给对应的页面传递参数，让Tabbar切换后的页面做一些特殊的展示，但是wx.switchTab 无法传递参数。

> ##### 解决办法 
> - 此时我们可以使用wx.reLaunch方法传递参数，关闭所有页面，打开需要的Tabbar页面

```javascript
      wx.reLaunch({
        url: path + "?tab=1",
      })
```

> - 在对应的页面接收参数

```javascript
    Page({
      onLoad (option) {
        console.log(option.tab)
      }
    })
```



