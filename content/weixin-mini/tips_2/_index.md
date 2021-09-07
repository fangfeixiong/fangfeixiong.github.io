---
layout: single
title:  "微信小程序开发Tips_2"
date:   2021-08-31
categories: 微信小程序开发
toc: true
---

#### 微信小程序分享到Android朋友圈的Beta版功能的使用问题

- 现在做的小程序的项目详情页引用了很多**第三方的组件**，导致分享到朋友圈的落地页无法完整显示，因为微信分享到朋友圈功能只能**支持部分微信小程序的原生组件**(详情见: [微信官方文档分享到朋友圈Beta](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/share-timeline.html)) 
  <br/>
- 分享到朋友圈的页面路径是固定无法更改的，从**页面A**唤起的分享到朋友圈，朋友圈进入的落地页就展示的**页面A**的内容，点击**前往小程序**也会进入**页面A**。


> 那么如何将很复杂的页面分享到朋友圈，又能解决朋友圈进入的落地页按照预期的显示，不会错乱了。我做了如下一系列操作，这个思路供大家参考，如果有更好的方法大家可以给我留言哦！

> #### 第一步
> 能够唤起朋友圈的分享，我习惯在onLoad的时候调用 **[showShareMenu](https://developers.weixin.qq.com/miniprogram/dev/api/share/wx.showShareMenu.html)** 方法，代码如下:
> 注意 **shareTimeline** 只对Android起作用。

```js
   onLoad: function(options) {
        //同时给分享菜单加入 分享给朋友 和 分享给朋友圈
        wx.showShareMenu({
            menus: ["shareAppMessage", "shareTimeline"]
        });
   }
```

> #### 第二步
> 实现页面时间处理函数 **[onShareTimeline](https://developers.weixin.qq.com/miniprogram/dev/reference/api/Page.html#onShareTimeline)** ,  这个函数是返回自定义的分享内容, 分享的标题、自定义的额参数和分享的图片等。
> <br/>*1.* 返回的 **query** 属性是页面路径携带的参数，不是页面路径哦，之前我这里理解错误，导致做了很多无用功。
> *2.* 这里的 **query** 我携带了两个参数:
> *target_id*: 分享的页面需要的参数，用来请求详情数据
> *is_timeline*: 用来判断是否是朋友圈加载的落地页或者 **前往小程序** 按钮进入的
> 
> 具体代码如下:

```js
   onShareTimeline: function() {
       return {
           title: '分享标题',
           query: 'target_id=1&is_timeline=1',
           image: 'https://path.jpg',
       }
   }
```

> #### 第三步
> 修改 **data** 和 **onLoad** 函数
> 1. 给 **data** 中加入 is_timeline 属性，用来判断是小程序环境 还是 从朋友圈加载或进入的

```js
 Page({
   data: {
       is_timeline: 0
   }
 })
```
> 2. 在 **onLoad** 函数中，获取内容详情id 和 是否是朋友圈的参数

```js
   onLoad: function(options) {
        //同时给分享菜单加入 分享给朋友 和 分享给朋友圈
        wx.showShareMenu({
            menus: ["shareAppMessage", "shareTimeline"]
        });
        //获取是否是朋友圈的参数
        if(options.is_timeline) {
            this.setdata({is_timeline: options.is_timeline})
        }
        //获取内容
        getInfo(options.target_id)

   },

   //获取信息内容
   getInfo(info_id) {

   }
```

> #### 第四步
> 修改WXML文件，将朋友圈落地页需要展示的效果布局和信息页原本的布局区分开，而且朋友圈落地页效果使用微信开发文档支持的组件。

```html
<view wx:if="{{is_timeline}}">
    信息的朋友圈落地页展示内容
      <view bindtap="tapTimelineDetail">前往详情</view>
  </view>
</view>
<view wx:else>
    信息的正式版本内容
</view>
```
> #### 第五步
> 增加去详情的点击事件，点击前往详情后可以选择切换显示 或者使用 **wx.redirectTo** 关闭所有，跳转到新的详情页


```js
   tapTimelineDetail: function(event) {
       this.setData({is_timeline:0})
       //或者
       //wx.redirectTo({url: path})
   }
```
> 到这里就算完成了，下面是实际使用的效果。如果有其他的解决方案可以给我留言哦！


{{< rawhtml >}}
<video id="video" controls="controls" preload="none" poster="tips2.jpeg" width="100%">
    <source id="mp4" src="tip2.mp4" type="video/mp4" >
</video>
{{< /rawhtml >}}
