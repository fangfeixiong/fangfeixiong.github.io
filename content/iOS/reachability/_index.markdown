---
layout: single
title:  "iOS判断是否连接网络"
date:   2017-06-12 20:40
categories: iOS开发
---

* content
{:toc}

####  遇到的问题
1. iOS10 以上的系统，第一次下载一个app的时候会出现这样一个问题。当用户还没有选择是否允许使用wifi和蜂窝网络的时候，如果这时候已经进入app的主界面，那么就会所有的数据访问失败，导致界面一片空白，没有任何数据。这时如果用户选择了允许访问wifi和蜂窝网络后，没有一个像允许通知的api，来通知app网络已经连接上了。

2. 为了解决这个问题，我们需要自己来监听网络环境是否有变化，如果监听到变化了，知道网络已经连接上，这时重新请求数据就能够获取到了，然后将app的主界面填充起来！

3. 监听网络环境的iOS没有现有的Api供调用，但是苹果官方有个Demo里面有一个对应的类Reachability [获取项目](https://developer.apple.com/library/content/samplecode/Reachability/Introduction/Intro.html)

4. 为了方便调用，我自己做了一个封装类  [封装类地址](https://github.com/PlayLive/Practice/tree/master/Reachability)

#### 调用方式

{% highlight objective_c %}

[[ReachabilityManager Instance] setHostName:"www.apple.com" complete:^{
    if([[ReachabilityManager Instance] canReachHttp])
    {
        [请求你自己的数据]();
        [[ReachabilityManager Instance] stop];
    }
}];
if([[ReachabilityManager Instance] canReachHttp])
{
    [请求你自己的数据]();
    [[ReachabilityManager Instance] stop];
}else
{
    [[ReachabilityManager Instance] start];
}
{% endhighlight %}
