---
title: 收藏地址的产品设计与开发实战
date: 2019-08-19
---

想到的第一个工具，叫做：收藏地址。请忽略小工具的名称，因为都是我随意起的，并没有什么含义，一个名字而已。

其实，这是一个很早就想到的需求，结合微信提供的地图 API，使得这个小工具的开发变得尤其简单。

### 需求来源
我有使用多个地图的习惯，例如：高德地图，腾讯地图以及百度地图。长途跨城开车，我通常使用高德地图，因为它的用户体验较好；短途开车或是查找商铺地址，我通常使用百度地图，因为它的商铺数据相对完整。

收藏地址的功能，应该算是强需求了，每一个地图软件都有这个功能。那么，做这么个小工具，想要满足什么需求呢？主要有两点：

一是，针对多地图用户，找一个统一的地方收藏地址。对我来说，这是一个痛点需求，记得第一次去处理交通违章的时候，花了一些时间，找到了处理大厅，于是，顺手就打开一个地图软件收藏了地点。然后，中间隔了好长时间，第二次去的时候，打开地图，明明记得收藏过的，可是找不到了，后来回想原来在另外一个地图软件里。

二是，提升打开效率。收藏夹的功能虽然是强需求，但是，对于地图软件来说，它并不是一个优先级较高的功能。往往入口藏得比较深，再加上某些地图软件的开屏广告，快速正确找到那个曾经收藏的那个地址，然后导航，不是一件轻松的事情。

这个小工具就在这样的需求背景下，诞生了。

### 产品思路
设计小工具的主要理念在于简单快捷，所以，这个小工具，总体来说，就是一个列表。列表具有添加、删除等功能。主要实现两个功能点就可以了：

一是，添加地址的时候，默认在地图中显示当前位置，便于快速收藏地点，同时，需要具备搜索地址的功能，可以提前收藏常用地址。
二是，点击列表中的地址，在地图中显示地址以及周边，同时，出现导航按钮，点击弹出导航菜单，可选择任意一个导航软件，便于快速导航。

恰好，这两个功能点，微信小程序已经给你封装好了，开发过程真是再简单不过了。这也是我把它作为第一个小工具的主要原因。

### 开发实战
微信开发的基础内容，暂且先不介绍了，主要介绍关键代码部分。

![](./_image/address.JPG)

首先前台页面就是一个列表，wxml 部分代码如下：

```html
<block  wx:for="{{ lists }}" wx:key="*this" wx:for-item="address" wx:for-index="index">
    <view class="weui-media-box weui-media-box_text wrap" bindtap='gotoLocation' data-index="{{ index }}" data-latitude="{{ address.latitude }}" data-longitude="{{ address.longitude }}" data-name="{{ address.name }}" data-address="{{ address.address }}" bindlongpress='editLocation' bindtouchstart='touchstart' bindtouchmove='touchmove' bindtouchend='touchend'>
        <view class="weui-media-box__title weui-media-box__title_in-text">{{ address.name }}</view>
        <view class="weui-cell__ft weui-cell__ft_in-access" style="bottom: 6px"></view>
        <view class="weui-media-box__desc">{{ address.address }}</view>
    </view>
</block>
<button class="weui-btn" type="primary" bindtap='add'>添加常用地址</button>
```

添加按钮上，绑定一个 add 的事件，在 js 中代码如下：

```js
add: function(){
    var that = this;
    var tempList = that.data.lists;
    wx.chooseLocation({
      success(res) {
        var address = {
          name: res.name,
          address: res.address,
          latitude: res.latitude,
          longitude: res.longitude
        }
        tempList.push(address);
        that.setData({
          lists: tempList
        });
        wx.setStorage({
          key: "lists",
          data: that.data.lists
        })
      }
    })
},
```

这里主要调用了微信小程序地图 API，[wx.chooseLocation(Object object)](~https://developers.weixin.qq.com/miniprogram/dev/api/location/wx.chooseLocation.html~)，选择地址成功，将会返回地址的名称、详细地址、纬度以及经度，四个属性值。

另外，使用 wx.setStorage() 将列表数据保存在缓存中，下次进入页面的时候，可以先读取缓存，于是，我们在页面的 onLoad 中添加 wx.getStorage() 方法的调用。

```
onLoad: function (options) {
    var that = this;
    wx.getStorage({
      key: 'lists',
      success(res) {
        if(res.data.length !== 0){
          that.setData({
            lists: res.data
          });
        }
      }
    });
},
```

当我们点击列表中某一个地址的时候，弹出地图，显示导航按钮。这个功能看似比较麻烦，其实同样是一句代码就能搞定，因为微信小程序官方提供了这样的 API，[wx.openLocation(Object object) | 微信开放文档](~https://developers.weixin.qq.com/miniprogram/dev/api/location/wx.openLocation.html~)，在视图中绑定了 gotoLocation 的方法，js 中代码如下：

```js
gotoLocation: function(e){
    var latitude = e.currentTarget.dataset.latitude;
    var longitude = e.currentTarget.dataset.longitude;
    var name = e.currentTarget.dataset.name;
    var address = e.currentTarget.dataset.address;
    wx.openLocation({
      latitude,
      longitude,
      name,
      address,
      scale: 15
    })
},
```

到这里，收藏地址小工具的关键功能就完成了，另外，补充上「删除」以及「置顶」等功能，就几乎完成了地图中收藏夹的功能了。

