---
title: 腾讯云功能
---

## 图像安全审核
图片审核前，可以先用wx.compressImage进行图片压缩。

这里为了方便，均已云开发中的云调用来作为基础，大部分功能其实也支持HTTP API的调用。


| 名称                  | 额度    |    数据类型         |
| :-------------------- | :-------: | :-------: |
| 图片安全审核<br />云开发扩展能力 | 2000张/日 |      云存储fileId中图片绝对地址       |
| imgSecCheck<br />开放能力 | 免费 |      Buffer       |
| imgSecCheck<br />服务市场 | 365天内免费10000次 |       ImageUrl、ImageBase64       |
| 违规内容识别 | 每月10000次（估计）  |        ImageUrl、ImageBase64       |
| 图片内容安全<br />T-Sec 天御| 30天内免费10000次 |       ImageUrl、ImageBase64       |


### 图片安全审核，来自云开发扩展能力
这是云开发最新推出的非常好用的人工智能能力，能够完成对图片安全审核的大部分需求。

**使用步骤：**
1. 上传图片到云存储
2. 云函数调用`图片安全审核，来自云开发扩展能力`进行校验
3. 如果是违规图片，可能还需要从云存储上删除，以免占用不必要的空间

### 开放能力 imgSecCheck
接口文档 https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/sec-check/security.imgSecCheck.html

在实际使用中发现的大问题，图片Buffer大小超过100k，可能就报图片太大的错误了。有的小伙伴说，10k都可能会报。

### 服务市场 imgSecCheck
接口文档：https://developers.weixin.qq.com/community/servicemarket/detail/000a246b6fca70b76a896e6a25ec15

根据 `返回值及示例 请见 https://cloud.tencent.com/document/api/865/35473#3.-.E8.BE.93.E5.87.BA.E5.8F.82.E6.95.B0`可以看出，其功能是基于`T-Sec 天御 图片内容安全`进行的封装，但允许试用1年。

### 违规内容识别

这个功能不仅支持图片地址，还支持图片Base64数据，也就是说无需将图片上传到图片服务即可进行识别。

> 这也是我目前在使用的方案，因为我的图片不大，并且当用户在确认需要保存图片后，我才会将图片上传到云存储上。

**我的步骤为**

1. 选择图片后，图片会被裁剪为600x600
2. 使用 wx.compressImage()将图片的质量从 `100` 降为 `10`
3. 将图片数据格式转换为Base64
4. 使用`违规内容识别`进行图片安全审核

> PS：腾讯云是将这个功能给雪藏了，我自己扩展了其`tencentcloud-sdk-nodejs`来做到的。

### T-Sec 天御 图片内容安全
接口文档：https://cloud.tencent.com/product/ims
这个的花费是最大的，我这里就不多讲了。



## 图像处理
图片智能裁剪或压缩
使用腾讯云功能完成小程序或web端做的不够好的

## 图片盲水印

明水印
盲水印
也是可以制作vip功能，充值后去水印


## 图片翻译

将图片中描绘的场景描述出来
谷歌有强大的，
腾讯云的这个也不错，够用