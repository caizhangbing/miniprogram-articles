---
title: 一个列表就可以是一个小程序
date: 2019-08-19
---

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

```js
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