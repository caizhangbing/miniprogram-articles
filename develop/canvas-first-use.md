---
title: 初次使用 Canvas 画布
---


列表页就不多说了，下面主要介绍一下，表单页以及分享页。表单页面有两个字段，一个是文本框，另一个是日期选择器。部分代码如下：

```html
<view class="cu-form-group margin-top">
  <view class="title">名称</view>
  <input placeholder="结婚/生日/高考/..." bindchange="inputChange"></input>
</view>
<view class="cu-form-group">
  <view class="title">日期</view>
  <picker mode="date" value="{{**date**}}" bindchange="dateChange">
    <view class="picker">
      {{**date**}}
    </view>
  </picker>
</view>

<view class="padding lp-bottom">
  <button class="cu-btn block bg-{{**mainColor**}} round lg shadow" bindtap="add">保存</button>
</view>
```

保存按钮绑定了 add 事件函数，点击保存按钮的时候，将数据写入缓存中，然后跳转到列表页。

```js
add: function(e){
  var title = this.data.title;
  var date = this.data.date;
  if (!title || !date) {
    wx.showToast({
      title: '名称不能为空',
      icon: 'none',
      duration: 2000
    })
  } else {
    var day = {
      title,
      date
    };
    wx.getStorage({
      key: 'days_list',
      success: function (res) {
        res.data.push(day);
        wx.setStorageSync('days_list', res.data);
        wx.navigateBack({
          delta: 1
        })
      }
    })
  }
},
```

上述代码做了一个简单的校验，然后保存数据就跳转到首页，所以这里使用了同步的方式保存数据，否则列表页上的数据不能及时显示。

接下来就是分享卡片的功能了，由于微信官方对分享功能的限制，所以，针对分享，几乎都是通过保存图片的方式实现。保存图片需要使用 canvas 画布，然后才能调用官方 API 实现。首先看下页面代码： 

```html
<canvas style="width: 100%; height: 200px;" canvas-id="firstCanvas"></canvas>
<view class="cu-form-group">
  <view class="title">是否显示目标日</view>
  <switch class="green radius sm" checked bindchange="switchChange"></switch>
</view>
<view class="lp-twoBottomBtn">
  <button class="cu-btn bg-green round lg" open-type='share'>发给好友</button>
  <button class="cu-btn bg-red round lg" bindtap="saveToImg" loading='{{ isLoading }}'>保存卡片</button>
</view>
```

这里添加了一个自定义的功能（是否显示目标日），实现原理就是，clear 后重新绘制。代码如下：

```js
switchChange: function(e){
  var value = e.detail.value;
  if(value) {
    clearDate();
    drawCard(this.data.title, this.data.date, this.data.number, this.data.isPast);
  }else{
    clearDate();
    drawCard(this.data.title, 'xxxx-xx-xx', this.data.number, this.data.isPast);
  }
}
```

这里定义了两个方法，一个是 clearDate()，一个是 drawCard()，传入固定的参数绘制图形，具体绘制的代码就不贴了，又长又臭，主要就是通过 `wx.createContext` 获取 context 对象后，画框，画字，画图片。

另外，绘制完卡片后，有一个保存到相册的功能，这个直接使用官方提供的 API 就可以实现了。

```js
saveToImg: function () {
  var that = this;
  this.setData({
    isLoading: true
  });
  wx.canvasToTempFilePath({
    canvasId: 'firstCanvas',
    fileType: 'jpg',
    quality: 1,
    success(res) {
      wx.saveImageToPhotosAlbum({
        filePath: res.tempFilePath,
        success: function (res) {
          that.setData({
            isLoading: false
          });
          wx.showToast({
            title: '保存成功'
          })
        }
      });
    }
}
```